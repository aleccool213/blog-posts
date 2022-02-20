## How Learning Elixir Made Me a Better Programmer 🥃

After getting comfortable with a couple programming technologies, developers usually stop there; your job and the systems you maintain may all be in one or two languages. You start using similar patterns again and again to solve the same problems. Elixir, a relatively new programming language, opened my eyes to new techniques which broke this stagnant thinking. Learning a new programming language can introduce you to techniques you never would've come across using your existing technologies. It expands your toolbox when it comes to designing new systems. Imagine a carpenter being stuck to a certain set of tools for years, they would be limited in what they could build. After learning programming languages for years (school, contract-work, co-ops, etc), it was refreshing to step away from a mindset focused on getting it done as fast as I could. No timelines telling you what velocity to learn at and no peers depending on you to finish what you were working on. I find that in this relaxed setting, it's easier to digest larger cognitive loads.

<table class="image">
   <caption align="bottom">E.g. of pattern matching. This and many other features of the language make it expressive and easy to read.</caption>
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362994/elixir-post/pattern.png" alt="pattern-matching-example"/></td></tr>
</table>

### Quick Facts for the T.L.D.R. in you

- Elixir is simply syntax on top of Erlang, the battle-tested language built on top of the BEAM VM

- The syntax is similar to Ruby so learning the syntax is easy and quick, especially for developers familiar with it

- Did I mention it's FUNCTIONAL! (Pure, functional programming IMO is worth the investment cognitively, <a href="https://medium.com/making-internets/functional-programming-elixir-pt-1-the-basics-bd3ce8d68f1b" target="_blank" >hit this link</a> for how Elixir utilizes it)

One of the benefits of learning a recently-created programming language is that it's built on top of existing best practices. This happens when the creators spend time thinking about what problems other developers face regularly. "State management is hard", "it's hard to have zero time deployments of new code", "it's hard to maintain my systems", something every developer thinks. Elixir wants to make these problems less hairy and does so using functional methodologies wrapped around a VM which puts distributed/concurrent programming as a first-class citizen.
Elixir for example was built by developers who saw the productivity of the Ruby syntax, the maintainability of functional programming and the scalability of Erlang. These features of the language make it a compelling showcase of what a language recently built can be, as showcased in the pattern matching example above.

> Elixir for example was built by developers who saw the productivity of the Ruby syntax, the maintainability of functional programming and the scalability of Erlang.

### Wires connecting to wires

<table class="image">
   <caption align="bottom">OTP in the anime-flesh</caption>
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362986/elixir-post/telephone_pole.jpg" alt="otp-wires"/></td></tr>
</table>

The rock solid foundation of Elixir is built on top of a library named <a href="https://en.wikipedia.org/wiki/Open_Telecom_Platform" target="_blank" >OTP</a>. OTP is an elegant way to handle all of the problems that arise in distributed programming, think work across nodes, handling async messages, etc. It not only is a library of functions but also a paradigm to work within. This keeps things consistent across systems and large teams. Instead of a single process handling your entire app (think Node.js), many isolated processes make up an Elixir app. These processes communicate to each other using messages. This unlocks a lot of cool features, processes can now live across machines as messages can only be immutable, no pointers allowed.

The critic inside you will say the potential downfalls of using such a new language is that it isn't battle-tested. Usually this is a valid criticism, such is not the case for Elixir. The VM Elixir it's built on top of is hella old. The initial open-source release of Erlang was in 1998, and Ericsson was using it in-house for a long time before that. Used by telecom networks, these were critical services which could not afford to have downtime. For example, that's how the very cool <a href="https://github.com/edeliver/edeliver" target="_blank" >hot-code-release</a> feature came to be which enabled developers to release new Erlang/Elixir code without taking down servers.

### My Experience

<table class="image">
   <caption align="bottom" style="font-style:italic;">A candid photo of me reading Elixir in Action</caption>
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362980/elixir-post/bill_reading.jpg" alt="pattern-matching-example"/></td></tr>
</table>

Last year, a coworker invited me to join his book club. "Lets learn this new language." I had heard it was the new hotness so I said, "sure!". We would take a couple of hours every month to go over a chapter in the book, <a href="https://www.amazon.ca/gp/product/161729201X/ref=as_li_tl?ie=UTF8&camp=15121&creative=330641&creativeASIN=161729201X&linkCode=as2&tag=coffeedrive09-20&linkId=97d40dff77b7869475d6ee283c6501d2" target="_blank" style="font-style:italic;">Elixir in Action</a>. Initially, it was intimidating to join as I was vastly junior compared to the other members of the group but I gave it a shot. What followed was lots of great discussions and insight into topics I haven't dove into before. I am appreciating my former self for agreeing to join as not only did I learn a lot, I connected with coworkers in the company I would have never connected with otherwise. It helped me through Flipp's adoption of Event Driven Systems (think Kafka) by exposing me to good practices when managing state between processes. Keeping processes small, pure and functional is good-sound engineering practice and are the pillars of how Elixir works. I didn't need anything to build immediately or an assignment to finish, I learned for the joy of learning and got a lot out of it.

### Common comments and questions

<table class="image">
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362988/elixir-post/road_forward.jpg" alt="pattern-matching-example"/></td></tr>
</table>

> My team is not going to be happy that after learning 3 Javascript frameworks in the past week, they have to learn this.

Once you start building things that have to scale or need to handle millions of requests, your on-call tickets increase. The reason for this is usually you can't predict traffic at that scale, push notifications go out for a new feature and everyone starts hitting your API. How do you handle this currently, with something like Node or Ruby? You just increase your box numbers and then decrease them after the load is done. This gets expensive and developers should not just be throwing money at something to solve a problem. Erlang VM processes (different than the traditional process) are a fixed size, this is **mega**. To a degree, this essentially solves this problem. Knowing how much memory processes are, gives you god-like abilities. The VM can tell the server precisely how much memory it may potentially use. Instead of falling over and the box restarting, you could respond to the client with HTTP Status Code 429 for example. No more unexpected memory loads at 1AM waking up developers!

> Okay, this is dope, how are errors handled?

Errors are a first class citizen in Elixir. Processes are small and isolated so when an error is thrown, the entire app process doesn't have to dump it's stack, just the isolated process. When errors do happen, they are easier to debug as the process code is small (by Elixir convention). Processes are so small that every process gets a monitor (another OTP blessing), which can run some code when a process dies. An example monitor could restart the process for example so that it could accept more messages.

<table class="image" >
   <caption align="bottom">Everyone gets a monitor</caption>
   <tr><td style="text-align:center;"><img style="margin-bottom:0px;" src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362991/elixir-post/everyone_gets.gif" alt="pattern-matching-example"/></td></tr>
</table>

Also, it's very neat that there is a proposal for pattern matching in Javascipt. Obvious proof that everyone is drinking the ... wait for it ... _Elixir_.

<table class="image">
   <caption align="bottom">🚒</caption>
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362983/elixir-post/javascript_pattern_matching.png" alt="pattern-matching-example"/></td></tr>
</table>

#### The road forward

I hope this introduction shows you some of the powers of Elixir and encourages you to learn more. I just scratched the service of what is possible with the BEAM VM. I leave you with this graph showing Elixir's popularity on Stackoverflow compared to other popular languages:

<table class="image">
   <caption align="bottom">Perspective</caption>
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362996/elixir-post/trends.png" alt="pattern-matching-example"/></td></tr>
</table>

The trend is upwards but it still has a long way to go for becoming somewhat mainstream.

Moving forward, I plan is just to write more and more Elixir code and get more comfortable with it. HackerRank has Elixir as an environment so it has been a great resource to practice the syntax. One of the next things I want to do is start creating something in [Phoenix](https://github.com/phoenixframework/phoenix).

Another resource I used in my learning journey was the <a href="https://www.meetup.com/TorontoElixir/" target="_blank">Elixir Toronto Meetup Group on Meetup</a>.

## Reading resources

The book we read during the book club was called Elixir In Action. A very good book which goes through the entire language and its features, in detail. The beginning is quite slow but as you start to wrap your brain around syntax, it soon becomes super interesting.

<table class="image">
   <caption align="bottom">Elixir in Action</caption>
   <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362993/elixir-post/elixir_in_action.jpg" alt="pattern-matching-example"/></td></tr>
</table>

This is another book I started which is much more approachable. It's a fun book which goes over the main features of why Elixir is a compelling language. This is a heart-pumper as it really just skims the surface.

<table class="image">
     <caption align="bottom" style="text-decoration:underline;">The Little Elixir & OTP Guidebook</caption>
     <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1544362985/elixir-post/opt_guidebook.jpg" border="0" alt="" style="border:none !important; margin:0px !important;" /></td></tr>
 </table>


> Originally posted on <a href="https://blog.alec.coffee/elixir-better-programmer/" rel="canonical">my blog</a>.

> Like this post? Consider [buying me a coffee](https://www.buymeacoffee.com/yourboybigal) to support me writing more. 

> Want to receive quarterly emails with new posts? [Signup for my newsletter](https://mailchi.mp/f91826b80eb3/alecbrunelleemailsignup) 
