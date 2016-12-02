---
layout: post
title: "Greasing the Groove"
date: 2016-12-30 22:00:00
categories: scripting training
---

## Introduction

There is a theory from the well-known and, uh, interesting strength trainer Pavel Tstsouline that a very efficient way to get stronger is by a method he calls "greasing the groove".
The idea is that you perform the given exercise for a relatively few number of repititions (i.e. few enough that each rep is with perfect form) many times throughout the day.
In this manner, you get in a large total volume of work over the course of the entire day, where each rep is ingraining perfect form (as opposed to training to failure, or lifting in a concentrated session, where fatigue begins to accumulate).

It is a fun way of training stuff if you have the means and, since I work from home and have a home gym, it seems like something that would be very easy to experiment with.
However, I hadn't been doing this sort of thing regularly, mainly because it was a pain to track (as I just use a paper notebook) and without tracking, it's difficult to maintain motivation.

Since I've mainly been working on big projects for the last few months, this seemed like a good thing to write some little scripts for and get back in the habit of automating stuff. Thus begins my journey!

## The Plan

I'm using the [i3 window manager][i3m] on my current computer, so I thought it would be neat if I could make a widget that would go in my status bar to track my daily extra exercises.
I had to switch to using [i3blocks][] instead of the standard i3bar to add custom blocks, but that was as simple as installing the program & changing a single line in my `.i3/config`.
Now, it was time to write some code.

I've been [writing lots of Clojure][braid] for the last few years (and I really enjoy it) but this sort of little script is not the domain that Clojure is good in.
I've also been [dabbling with Rust][octocat] and think it is very cool, but for something that I wanted just whip off quickly I decided to go with my old go-to -- Python.
I've also been reading [about Python 3 being great][eevee-py3] and I don't think I've actually used 3 before (although there aren't really many substantial differents between 2.7 and 3.5 that I've encountered).

## Part One

Whenever making something, I like to start by just doing the backend database stuff -- I am not a frontend kind of person and that's the part I enjoy the most.
I decided just using a sqlite database would be the easiest way to store the data - a CSV or plain-text might be more straightforward, but I thought that firstly, Python has a solid sqlite library, secondly this will save me from having to parse dates and stuff, and thridly this will enable me to do analytics and stuff in the future.

So, just making a script that can create a database, add a number of reps to an exercise, and see all the exercise for the current day, got me something like this (in a file in my `~/bin` named `grooving.py`):

{% highlight python %}
import datetime
import os
import sqlite3


def get_connection():
    return sqlite3.connect(os.path.expanduser("~/grooving.sqlite"),
                           detect_types=sqlite3.PARSE_DECLTYPES)


def init_db():
    conn = get_connection()
    c = conn.cursor()
    c.execute("""CREATE TABLE IF NOT EXISTS exercises (
        date DATE,
        exercise TEXT,
        count INTEGER)""")
    conn.commit()
    conn.close()


def increment_count(exercise, count):
    today = datetime.date.today()
    conn = get_connection()
    c = conn.cursor()
    c.execute("SELECT count FROM exercises WHERE date = ? AND exercise = ?",
              (today, exercise))
    cur = c.fetchone()
    if cur is None:
        c.execute("""INSERT INTO exercises (date, exercise, count)
                  VALUES (?, ?, ?)""", (today, exercise, count))
    else:
        c.execute("""UPDATE exercises SET count = ?
                  WHERE date = ? AND exercise = ?""",
                  (cur[0] + count, today, exercise))
    conn.commit()
    conn.close()

def todays_exercises():
    today = datetime.date.today()
    conn = get_connection()
    c = conn.cursor()
    c.execute("SELECT exercise, count FROM exercises WHERE date = ?",
              (today, ))
    exercises = c.fetchall()
    conn.close()
    if len(exercises) == 0:
        print("Nothing so far")
    else:
        print(' '.join("[{0[0]}]: {0[1]}".format(r)
                       for r in exercises))


if __name__ == '__main__':
   init_db()
   todays_exercises()
{% endhighlight %}

(Hard-coding database paths, 'cause it didn't seem worth the effort to make it configurable at this point)

## Part Two

The next step was to make this display in my status bar.
This was pretty straightforward, thanks to i3blocks' [solid docs][adding-i3-blocks].
I just added a block to my `~/.i3blocks` like this:

{% highlight cfg %}
[grooving]
label=GtG
command=$HOME/bin/grooving.py
interval=60
signal=12
markup=pango
{% endhighlight %}

This makes the output of my script show up in my status bar ![status bar output](/images/grooving/status_bar.png)

Note that in the block, I both gave it an interval to refresh on and a signal, so it will run the script to re-update the text in the bar both every sixty seconds *and* whenever i3blocks recieves a signal with value 12 (i.e. SIGUSR2).
The docs recommend against this, but I needed to do so in this case:
I want it to update on demand, when we add a new exercise, but I also don't want to see yesterday's information at the beginning of a day before I've tracked anything.

Now, I can use python in a terminal to add a new exercise and then run `$ pkill -SIGUSR2 i3blocks` to have the display update.

## Part Three

While I can manually add things in the console, the whole point of this exercise was to make it frictionless, so I wanted something more ergonomic.
Since i3blocks can tell the script when it recieves a mouse click (by invoking the script with the environment variable `BLOCK_BUTTON` set to a number indicating which button was pressed), I wanted some way to have the script present a GUI to let me quickly type in the exercise and number of reps.
Doing some reading around, it seemed like the easiest way to do this in Python is using `tkinter`.
I managed to make a very simple little GUI popup using the [tkinter docs][tkinter].

![the simple input program](/images/grooving/popup.png)

No buttons, it just adds when you hit return in a text field.

(The beauty of just making tools for yourself is you can make interfaces as crappy as you want as long as they work for you.)

The code looks like this (in `~/bin/grooving_add.py`):

{% highlight python %}
import tkinter as tk
import grooving


class Application(tk.Frame):

    def __init__(self, master=None):
        super().__init__(master)
        self.title = "Add Dialog"
        self.pack()

        self.create_widgets()

    def create_widgets(self):

        self.exercise_entry = tk.Entry()
        self.exercise_entry.pack()
        self.exercise = tk.StringVar()
        self.exercise.set("")
        self.exercise_entry["textvariable"] = self.exercise
        self.exercise_entry.bind('<Key-Return>', self.maybe_save)

        self.count_entry = tk.Entry()
        self.count_entry.pack()
        self.count = tk.IntVar()
        self.count.set(0)
        self.count_entry["textvariable"] = self.count
        self.count_entry.bind('<Key-Return>', self.maybe_save)

    def maybe_save(self, event):
        exercise = self.exercise.get()
        count = self.count.get()
        grooving.increment_count(exercise, count)
        grooving.todays_exercises()
        self.master.destroy()


def main():
    root = tk.Tk()
    root.title("Add Dialog")
    app = Application(master=root)
    app.mainloop()

if __name__ == '__main__':
    main()
{% endhighlight %}

Now this can be run manually; I can type the exercise name & number of reps in, it will be added to the database, and when I run `pkill -SIGUSR2 i3blocks`, I'll see the updated exercise information in the status bar.

However, I wanted to have this run when the bar is clicked.
This is done simply by adding `import grooving_add` to `grooving.py` and changing the `main` block at the bottom to look like this:

{% highlight python %}
if __name__ == '__main__':
    if os.getenv("BLOCK_BUTTON"):
        grooving_add.main()
    else:
        init_db()
        todays_exercises()
{% endhighlight %}

The final step I needed was to make the popup window appear as a "floating" window, so it looks like a little popup, instead of a full-pane app.
This is done by adding a line to my `~/.i3/config` like this: `for_window [title="Add Dialog"] floating enable`.

## Part Four

Yay!

![it works](/images/grooving/demo.gif)

  [i3m]: http://i3wm.org/
  [i3blocks]: https://github.com/vivien/i3blocks
  [braid]: https://github.com/braidchat/braid
  [octocat]: https://github.com/braidchat/octocat
  [eevee-py3]: https://eev.ee/blog/2016/07/31/python-faq-why-should-i-use-python-3/
  [adding-i3-blocks]: https://github.com/vivien/i3blocks/wiki/Writing-Your-Own-Blocklet
  [tkinter]: https://docs.python.org/3.5/library/tkinter.html#the-window-manager
