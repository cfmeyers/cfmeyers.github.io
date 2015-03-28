---
layout: post
title: "Things I Recently Learned About Heroku"
date: 2015-03-27T17:45:24-04:00
---

Learned yesterday that `rake assets:precompile` is considered harmful.  I've done this several times to get a Rails app working on Heroku, generally after following advice on Stack Overflow.  It does indeed precompile your assets, and frequently this causes the error to go away.  It's bad because it's treating the symptoms, not the real problem.

The real reason I'm erroring out when I try to push to Heroku is Heroku is trying to run `rake assets:precompile` and is failing, usually because my CSS is missing a semicolon or bracket.  

If my CSS is wrong, how come it still renders in the browser on localhost?  I'm guessing that Heroku is stricter about these things than my browser and is insisting on correct code.

The problem with running `rake assets:precompile` manually is it fills my public directory with precompiled CSS.  When I make a change to CSS, the Rails dev server (sometimes?) serves up the assets in the public folder, which won't reflect the changes I made.  This can be remedied by running `rake assets:precompile` again (and again and again), but better to just not go down that route.

##Faith-Based Deployment

The larger probem is I'm treating large chunks of the deployment process as magic.

I frequently encounter errors I don't immediately understand while coding.  I check, you guessed it, Stack Overflow.  But the simalarity ends there.  I see that I was calling the function with the wrong number/order of arguments, a `nil` had crept in, I didn't require the right library, or any other of a several other patterns.  Tracking down an errant `nil` might be tedious, but it isn't magic. 

Contrast that with deploying.  The weekend I deployed my first Rails app to a Digital Ocean VPS I had open, at any given time: 

-  3 or 4 different tutorials 

-  several contradictory blog posts

-  some guy's github commit from 2012

-  wikipedia (yes, wikipedia)

It was a frustrating process, made all the more frustrating because I know I'll be repeating it for my next deployment.  I *am* learning to deploy, but looking at the above `rake assets:precompile`, I worry I'm not learning fundamentals or core concepts, but rather Cargo Cult magic formulas.

Learning new things with code is a joy because of the fast feedback loop.  Not so with deployments.  They're widely spaced apart, frequently involving different pieces.  Even when the pieces themselves are the same, the state of the server varies.  The different parts interact with each other in an opaque way (opaque to me, at least).  

Tracing a problem with a deployment is a lot like tracing an error in a program that egregiously violates the Law of Demeter...is that error really happening on line 39 of `foo.rb`, or do you need to prepare yourself for a night of code spelunking?  Am I getting that "Bad Gateway" screen because Passenger is improperly configured, or because Postgresql hasn't got the right permissions, or because Nginx isn't running, or... 

And to top it all off, deploying is high-pressure.  You just want the damned thing to work and be done with it.

So no answers here, just have to keep pushing forward.  State sucks.  


