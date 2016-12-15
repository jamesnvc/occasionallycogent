---
layout: post
title: "Exploring Io"
---

I first heard about the [Io programming language][iolang] quite a while ago -- I believe I saw it in [Steve Yegge's][steveyegge][^yegge] sidebar when I was still in high school and just starting to get interested in different programming languages.
Recently, it came up again on [Lobsters](https://lobste.rs) and I decided to have another look at it.

The first programming language I really got into when I first started learning to code was Scheme, so I really appreciate the minimalist "pure" feel of Io.
The way that everything is just sending messages to objects, with "syntax" like assignment operators just being macros for messages (e.g. `foo = bar` is sugar for `updateSlot("foo", bar)`) really appeals to me.
I also have a soft spot for the RPN-like notation that Io code tends to have -- the first open-source project I contributed to was [the Factor programming language][factor], so it's a nice dose of nostalgia for me.

To try make something real in Io, I decided to write a little command-line client for [Pinboard][pinboard].
Pinboard has a very simple API, so it seemed like a good target for making a little toy program.

Confusing things I ran in to:

  - `URL with("https:/...") fetch` didn't work -- default URL thing apparently doesn't support SSL.
    The fix was easy enough though, just used the `HCConnection` class & friends, which are mentioned in the reference.
  - I got a little thrown off by newlines being statement separators -- it makes sense, but I think I was thinking of it as too much of a concatenative language.
  - I like that I can see the slots objects have, but I'd love if I could also see docs for methods in-repl too (like Clojure's ability to look up vars' docstrings)
  - Trying to figure out how to parse CDATA from XML -- there is apparently an XmlReader module, but no documentation, I can't figure out how to make SGML do something sensible with CDATA content
  - Eventually figured out, through trial-and-error, how to make the XML reader descend to the node of interest and read the CDATA.
    Unfortunately, it didn't decode the XML-entities, so I need to figure out how to do that now
  - There's a method on `Sequence` called unescape, which says it "replaces escape codes with escape characters", doesn't seem to do what I want
  - There's a method on `Sequence` called `replaceMap(aMap)`, which I *think* will let me give keys & values, where the keys are replaced with the values.
    Kinda annoying that the class it says I can use isn't mentioned in the guide at all (although it is in the reference at least).
  - I wanted to try to give it a little GUI, as Io apparently has bindings for various graphics frameworks, but there doesn't seem to be any sort of documentation
  - It looks like there's some sample code in the repo, but apparently I can't load OpenGL because my Io doesn't have Cairo?
  - I tried building Io from source myself, but Cairo still isn't linked in & I don't know why
  - Thought about making a terminal UI with curses, but the addon is commented out & apparently source files are missing

I think Io is a neat language, but I found it a little annoying to try to make standalone things with it.
Next time I need an embedded language, I think it'd be fun to try using Io instead of Lua though.

Here's the little pinboard client I made:

{% highlight io %}
Post := Object clone do(
  url ::= nil
  description ::= nil
  time ::= nil

  fromXML := method(xml,
    url = xml attributes at("href")
    description = xml attributes at("description")
    time = Date clone fromString(xml attributes at("time"),
                                  "%Y-%m-%d'T'%H:%M:%S%Z")
    self
  )

  asString := method(
    "Post: url=#{url} desc=#{description} time=#{time}" interpolate
  )

)

SGMLElement tagText := method(tag, elementsWithName(tag) first allText)

Sequence xmlDecodeEntities := method(
  self asMutable replaceMap(Map clone with("&quot;", "\"", "&amp;", "&"))
)

Note := Object clone do(
  id ::= nil
  title ::= nil
  hash ::= nil
  created ::= nil
  updated ::= nil
  length ::= nil
  body ::= nil

  fromXML := method(xml,
    id = xml attributes at("id")
    title = xml tagText("title")
    hash = xml tagText("hash")
    created = Date clone fromString(xml tagText("created_at"), "%Y-%m-%d %H:%M:%S")
    updated = Date clone fromString(xml tagText("updated_at"), "%Y-%m-%d %H:%M:%S")
    length = xml tagText("length")
    self
  )

  fetchBody := method(api,
    if(body == nil,
      // TODO: body is wraped in [CDATA[[general]], should extract it
      fetchedXml := api apiRequest("/notes/#{id}/" interpolate)
      body = self extractTextCdata(fetchedXml)
    )
    body
  )

  extractTextCdata := method(xmlText,
    reader := XmlReader clone
    reader parseString(xmlText)
    reader next
    reader moveToElement
    while(reader name != "text", reader read)
    reader readString xmlDecodeEntities
  )

)

Pinboard := Object clone do(
  authToken ::= nil

  apiURL := "https://api.pinboard.in/v1"

  apiRequest := method(path,
    // TODO: error handling
    HCRequest with(
      HCUrl with("#{apiURL}#{path}?auth_token=#{authToken}" interpolate)
    ) connection sendRequest response content
  )

  recent := method(
    apiRequest("/posts/recent") asXML elementsWithName("post") \
      map(post, Post clone fromXML(post))
  )

  notes := method(
    apiRequest("/notes/list") asXML elementsWithName("note") \
      map(note, Note clone fromXML(note))
  )

)
{% endhighlight %}

So using it looks something like

{% highlight io %}
doFile("pinboard.io")

pin := Pinboard clone

pin setAuthToken("pinboard-username:secret-token")

recentBookmarks := pin recent

notes := pin notes

notes println

firstNote := notes first

firstNote fetchBody(pin) println

firstNote println
{% endhighlight %}


  [iolang]: http://iolanguage.org/
  [steveyegge]: http://steve-yegge.blogspot.ca/
  [factor]: http://factorcode.org
  [pinboard]: https://pinboard.in

  [^yegge]: I owe a huge debt of gratitude to Steve Yegge: his [original blog](https://sites.google.com/site/steveyegge2/) was a huge influence on me when I first started programming and has really shaped the sort of developer I am today.
