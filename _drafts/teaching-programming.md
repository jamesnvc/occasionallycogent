---
layout: post
title: "Teaching Programming"
categories: programming teaching
---

How we teach programming has been something that I've been thinking about quite a bit lately.

  - I've been mentoring at a local [learn-to-code boot-camp][lighthouse] for about a year
  - My co-founder has also been mentoring & writing curriculum at the same place and we've both thought about [ways of teaching programming][cognitory] independently of the boot camp as well
  - My girlfriend is currently in a Master's program to become a teacher and (since her degree is in mechanical engineering) is often tapped to teach STEM subjects
  - My mother is a public-school principal and has been recently attending some workshops on learning coding for teachers

Something that I've come to believe is that we talk about teaching programming, we conflate two different things:
Teaching programming in the abstract (i.e. more computer science) and teaching how to make computer do stuff.

I think people that are already programmers tend to think about teaching programming as primarily from the computer science/theoretical angle and therefore that learning programming should start with learning things like big-O notation and getting a thorough grasp of the underpinnings of how computation and their language works.

While I think that those things are very important to learn and, if one really enjoys programming and wants to do it for its own sake, those are things that definitely should be learned.
However, when time is limited (for example, in a boot-camp environment) or when the learners are not necessarily people that want to be programming for its own sake, I think other things are more important.
One thing I've really noticed, for example, is that learning how to use the tools of a programmer is a not-insignificant challenge for people new to the task.

If one's goal is not so much to learn how to program in the abstract, but to make something that incidentally requires programming to make real, then wrestling with the tools will be a much bigger obstacle than not understanding monads.
The boot-camp I teach at is specifically to teach iOS development; teaching how to use XCode and git is far more valuable to the students than algorithms would be -- In fact, the lecture on data structures and algorithms doesn't come until the fifth week of the eight-week program!

Instead of trying to teaching students the basics, building up from fundamental algorithms in Scheme (which I would prefer!), writing things like parsers and compilers, we jump right in to making real apps that they can use to do real things.
I think that, by having them make tangible things (apps that one can interact with are much more exciting & satisfying that a command-line tool!) the students' interest and excitement is maintained; I always emphasize to my students that at the end of the course, they'll be well-positioned to *start* learning, if that's what they want.

I've started to emphasize the *if that's what they want* part more recently too.
I don't think that everyone needs to be a programmer, but being able to *do* programming can be very useful in other fields.
If some graduates of the boot-camp never follow my recommendations for algorithms books and just use the abilities that they've developed to make themselves better in some other field, I'm happy with that.
[John Siracusa][hypercritical], on a recent episode of [ATP][] talked about teaching kids to "drive the computer", which I think is a useful analogy:
I am fine if the people that get through the course don't become car mechanics, but if they get better and driving their car and are able to, like, do regular maintenance (I don't know enough about cars to flesh out this analogy), that's great!

Again, I don't want to diminish the importance of learning the fundamentals and building up -- that's how I learned! -- but not everyone has the luxury of all the time required to do so.
I think there's also a case to made for it being easier to appreciate the basics when you've seen the things that can be built atop them and used them in practice.

  [lighthouse]: https://www.lighthouselabs.ca/
  [cognitory]: https://github.com/cognitory
  [hypercritical]: http://hypercritical.co/
  [ATP]: http://atp.fm
