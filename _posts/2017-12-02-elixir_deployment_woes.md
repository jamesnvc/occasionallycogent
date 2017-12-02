---
layout: post
title: Elixir Deployment Woes
date: 2017-12-02
---

# The Problem

Things started when I finally finished up our [Gmail integration bot][mailbot] for [Braid][].
I had everything working locally (in which, by the way, [ngrok][] was very useful for testing the gmail webhooks) but then it was time to deploy it to the server.
I know I've been a little spoiled by working with Clojure, where deploying is just making an uberjar and throwing it up to the server, so I was worried that deploying would be some annoying thing that would require installing an Elixir & Erlang runtime on the server.
I was pleasantly surprised then, to find [Distillery][], a tool that let me build a simple tarball of the app, included a packaged-up erlang runtime.

This was great; I was able to package it up, upload it to the server, and it ran!

However, as soon as I tried to actually send a message to the bot, I got an error about the database schema not being found.
This made sense, as I hadn't created the database in prod.
Luckily, the Distillery documentation provided an [example][migration] for creating the database & running the migrations.
I was able to very easily adapt their code to add a migration script, which seemed to work.
I'm just using the Erlang Mnesia database, via ecto_mnesia, so everything is nicely packaged up and all under the purview of the same runtime as the rest of the app -- no worries about external database configuration.

The migration script ran and I could see the database files being created in the specified directory.
However...it still didn't work.

When the bot tried to run without the database present, it would give an error the first time it tried to query the database, saying that the users table didn't have an `id` column.
While this is technically true, it is a slightly misleading error, as the real problem is that the database doesn't exist.

After creating the database, the bot now gives an error from the same place in the app, this time saying that `Schema "users" does not exist`.

This is where I was stuck for two weeks.

I was utterly perplexed -- it seemed like it couldn't find the database, but the fact that I got a different error when the database exists versus when it doesn't seemed to indicate that wasn't the problem.
I was thinking that maybe the `mnesia` process wasn't being started properly or something, but I was able to use the (very useful!) `remote_console` command to get a REPL on the running process and see that `Application.started_applications/0` showed the `ecto`, `ecto_mnesia`, and `mnesia` applications all running.
Clearly, something different was happening when running the `migrate` command (that works!) versus starting the application.

The first clue, in retrospect, was that  entering the commands in the `remote_console` that the migrate script runs gave a strange error when trying to create the database:
`create_schema failed with reason {:nonode@nohost, {'All nodes not running', :nonode@nohost, :nodedown}}`.
I was pretty sure that `All nodes not running` was incorrect, because I could see the database application had been started and trying to start it manually gave an error indicating it was already running.
The `nonode@nohost` was very interesting though...that was the default name for an erlang node (I guess?) but the application should be running as `braidmail@127.0.0.1`.
So why this discrepancy?

# The Solution

Eventually, with the help of my co-founder Raf, we traced through the mnesia & ecto_mnesia code to determine exactly what was happening and realized that, although we're setting the `mnesia_host` in our config, the ecto_mnesia uses Confex to load configuration information.

Using Confex in our app turned out to be a bit annoying (the most recent version is 3.3.1, but ecto_mnesia uses 1.5), but we eventually were able to set it up to directly set the mnesia host in our Ecto repo's `init/2` callback.
This did something, since running `storage_up` in the remote console worked, instead of giving the "all nodes not running" error, properly said that the database was already created.
Even after that though, it still couldn't query the database, giving the same error about the schema not existing.

Since our database connection in the running application was working now, I was able to do more poking around.
The key clue was that, when running some mnesia info command in the remote console, it had some references to our old friend `nonode@nohost`.
Running `strings` on the created schema file showed a bunch of instances of `nonode@nohost`, confirming my suspicion that the migrate script was running as `nonode@nohost` and somehow baking that in to the database.
Figuring out how to make the migrate script run as the proper host name was not something I wanted to waste any more time on -- for whatever reason, just using an environment variable & Confex to set `mnesia_host` would make the script crash -- so we just put the important calls that the migrate script makes into our application's `start`.
Now, finally things work.

# Questions

I really wonder why this was so hard to figure out.
Is running a mnesia database in a deployed environment really such a rare thing?
I would think that, even if big things would use Postgres, this would be a common simpler option, instead of using SQLite.
I spent quite a while over the last two weeks searching for all manner of permutation of my problem and didn't see anything about mis-matched node names or whatever it was that was happening.

In the process of trying to track down exactly what the error was, Raf & I were also a little stymied by the Elixir/Erlang error handling model.
While it is nice & composable to write, the fact that it's just `{error, Reason}` return values being passed around means that the context of an error that occurred deep in the guts of a library kind of ends up being stripped of all context.
When faced with a rather generic error message, it would be pretty nice to have that be an actual stack trace.

Like my [previous encounter][gotcha] with Elixir debugging, I'm kind of disappointed.
It seems like another example of a pretty nice language, with very nice docs for the simple case, but zero on the more complicated, rubber-hitting-the-road kind of information.
Anything about mnesia node names being vital, or error messages that actually indicated what the real error was would have made this way less of an ordeal.

As it is, I'm feeling a little burned on Elixir.
Say what you will about Clojure's Java underpinings, that shit is documented to hell and back.
I also feel much more appreciative of exceptions -- they may be less transparent & composable, but the way they let the full context of the error be exposed is so incredibly valuable in this sort of situation.

  [mailbot]: https://github.com/braidchat/emailbot
  [Braid]: https://github.com/braidchat/braid
  [ngrok]: https://ngrok.com
  [Distillery]: https://github.com/bitwalker/distillery
  [migration]: https://hexdocs.pm/distillery/running-migrations.html#content
  [gotcha]: /2017/11/15/an-elixir-gotcha.html
