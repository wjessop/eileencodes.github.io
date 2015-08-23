---
layout: post
title: "There's More to User Experience Than Ease of Use: Lessons in Not Scaring Your Customers"
date: 2012-01-31
categories:
  - tips
  - user-experience
---

<p>This post is not really going to be Rails related. But everything in a Rails blog doesn't need to be about Rails, or Ruby, or coding at all.</p>
<p>I feel like Rails is an experience, in not being frustrated with everything and it's easier to map out an application. The things that get in the way of building something awesome, the time consuming, awful things like join queries and pluralization are done for you. You can focus on your application and it's beauty and structure.</p>
<p>Well, user experience is also an experience and I hate that it's so often that a website has me banging my head against a wall. Recently, I logged on the Sallie Mae website to check something with my student loan. Being that I have it set for auto-debit and that I was logging in on a weekend I was met with a front page telling me I was delinquent on my loan.</p>
<p>Now people who know me will know my reaction without telling you. I literally flipped a shit. Thinking my bank account information was lost or some other awful scenario (was my bank account hacked and emptied?? no...it wasn't phew) I searched through my account for an answer when the answer occurred to me.</p>
<p>Sallie Mae does not have a failsafe in place so that when you have auto-debit activated and the date of debit falls on the weekend it doesn't "figure it out" it instead tells you you didn't pay and that you better do it now.</p>
<p>Awesome. Please when building an application make sure you put in if else statements that check for things like weekends, and auto-debit. It will make your customers happy. No one wants a mini heart attack on Sunday evening, to log in the next day and see all is well.</p>
<p>User experience is more than making a site easy to use, and beautiful (both which Sallie Mae fail at). It's also about not scaring the crap out of your clients and consumers &mdash; or accepting 76 early acceptance students accidentally (I'm looking at you Vassar). When desiging a web application consider all the horrible things that can go wrong for the user, not just you the developer and put failsafes in place to prevent those things. You will have happier clients that will keep coming back again and again.</p>
<p>Frustration is never good for business.</p>
