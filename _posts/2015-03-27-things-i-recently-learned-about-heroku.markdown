---
layout: post
title: "Things I Recently Learned About Heroku"
date: 2015-03-27T17:45:24-04:00
---

Learned yesterday that `rake assets:precompile` is considered harmful.  I've done this several times to get a Rails app working on Heroku, generally after following advice on Stack Overflow.  It does indeed precompile your assets, and frequently this causes the error to go away.  It's bad because it's treating the symptoms, not the real problem.

The real reason I'm erroring out when I try to push to Heroku is Heroku is trying to run `rake assets:precompile` and is failing, usually because my CSS is missing a semicolon or bracket.  

If my CSS is wrong, how come it still renders in the browser on localhost?  I'm guessing that Heroku is stricter about these things than my browser and is insisting on correct code.

The problem with running `rake assets:precompile` manually is it fills my public directory with precompiled CSS.  When I make a change to CSS, the Rails dev server (sometimes?) serves up the assets in the public folder, which won't reflect the changes I made.  This can be remedied by running `rake assets:precompile` again (and again and again), but better to just not go down that route.


