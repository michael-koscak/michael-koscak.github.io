---
layout: post
title: "Lessons Learned from AI Coding"
draft: true
---

## Introduction
[Your intro here - set the scene about what vibecoding is and why you started]

When I used to live in Chicago I volunteered at Habitat for Humanity semi-regularly.  I remember being up on the roof and hammering in nails and asking the guides at the build site why they didn't provide nail guns.  We could do this work so much faster if we had a nail gun and I'm sure they could dig up someone who'd donate some.

The answer was simple - volunteers at Habitat are not pro-builders and while a nail gun also helps to build fast, they can also screw up the build way faster with it.  This is the most apt comparison I can make for using AI developer environments.  At first it feels incredible how much you can build quickly, but if you don't know what you are doing it can snowball into a disaster super fast.

Bullet ideas to cover:
- Define vibecoding (AI-assisted rapid development)
- What project(s) you built this way
- The initial euphoria of 10x productivity
- Teaser: why you're writing this post now

## 1. The Seductive Trap of Instant Gratification

[Your content about how it starts easy but snowballs]

Bullet ideas to expand on:
- The dopamine hit of seeing features materialize in minutes vs hours
- Examples of "wow I built X in 30 minutes!" moments
- The technical debt interest rate is brutal
- Security vulnerabilities you discovered later
- Specific example of a "simple" feature that became a maintenance nightmare
- The false economy of speed vs. quality
- How refactoring AI-generated code is often harder than writing from scratch

As someone who struggled learning to code in college, using these tools is incredibly fun.  At first use building a POC or initial version of something I was stunned how quickly I could turn idea into reality.  However it very quickly becomes apparent that thinking through the idea and how you structure the codebase is critical if you want to actually do anything with what you build.  Tech debt can explode with this and you need to design not only for the code to work but also in a way that you can maintain.

With my Echo & Chamber project I actually restarted the build from scratch 3 times.  The first time I tried to megaprompt "give me an app that monitors MSNBC and Fox News and creates a newsletter".  It actually kind of worked but not well enough.   After iterating a few times I realized that the best way would be to create multiple repositories for the different functions, and to skeleton the app design before creating it.  There are some apps online that help you do this but for me working with ChatGPT & Claude to create the spec worked well.  The key here is that you actually read and understand all of it, which transparently is tempting to not do because initially the AI will still probably build something pretty good and your problems won't become clear until later.

## 2. The Repository Revelation: Modular or Die

[Your content about breaking things into multiple repos]

Bullet ideas to expand on:
- Your evolution: monolith → attempted refactor → nuclear option (restart)
- Why separate repositories saved your sanity
- Specific example of how you split your project
- How modularity actually makes AI coding MORE effective
- The boundaries that make sense (data layer, API, frontend, etc.)
- Version control and deployment benefits
- How this applies to non-AI development too

For me the biggest thing that helped is to keep everything you can contained in a specific repo or file.  By 1) creating dedicated repos and 2) telling the AI how to structure the files I found that building became way more managable.  Instead of "Build feature X" my prompts always have some sort of "Build feature X and put it in a single file that is called as functions from our main app".  Of course what feature X is would also have a lot more detail, but doing it this way allows me to keep track.  It also helps that I can read function calls from main and diagnose what's happening, although asking Cursor how something works is usually how I start to edit.

## 3. The Planning Paradox: Structure Without Rigidity

[Your content about the nuanced approach to planning]

Bullet ideas to expand on:
- Why "just plan everything" is naive advice
- Your planning sweet spot: what to define vs. leave flexible
- Examples of over-planning that got thrown out
- Examples of under-planning that caused rework
- The "north star" approach - clear vision, flexible implementation
- How to document decisions when moving fast
- Planning for pivots: making your code pivot-friendly

Keeping in mind the ideas so far, planning becomes critical.  However there is this dualing issue of wanting to plan but also trying to be agile and generally I don't entirely know the full scope of what I'll build in the future.  This is where "vibe coding" falls apart and developer experience becomes necessary.  You really need to understand architectural and design patterns to steer the AI towards an architecture that works for what you want.

Additionally as I build I also find myself asking Cursor to document what it built in dedicated .md files.  When I go to edit stuff I can then feed in the MD file so the AI knows what exists, and then after making a code update we also update the MD file.  This trick really helped me as the codebase started to scale.  I even took it futher as I needed the codebases to interact and would create files to take between repos so the various codebases knew how eachother worked.  There are probably better ways to do this, but I am a hobbiest so I just find stuff that works for me quick & do that.

## 4. Practical Takeaways

Overall my takeaway is that non-developers are not going to thrive in a tool like Cursor.  I am sure pro-developers could talk about this a lot more than me, but my lense is a sales engineer who likes to stay up to date on tech & tinker with stuff.  For me it allows me to build more than I could ever imagine, but I know how to code.  I would definitely not say my software engineering skills are strong, but since I know enough about code architecture it allows me to build.

The biggest callout that I'd have for these types of tools is it totally changes the paradigm of development and you need to actually practice.  A rockstar dev who is handed AI coding for the first time would also probably struggle for a bit.  Most of the workflow does wind up becoming specific to your individual preference and can't generalize, so you won't get good unless you just actually spend time doing it.

## Conclusion

For me as an Engineer using AI has been incredible.  Something that has also been top of mind for me lately is all the AI doomerism in the news - ridiculous $$ amounts being thrown around in circular patterns and fear of a bubble.  I do oftentimes find myself talking about my work to non-tech people and I can sense skepticism of it, people are afraid of job loss and environmental issues from crazy AI datacenter builds.

I don't diminish those concerns, they are real problems.  But I do think for folks not in the tech industry they see a ton of AI slop being created, issues with deepfakes, and generally have a negative perception of the tech.  I hope that as an industry tech people can figure this out and move away from greed and promote the good use cases like using AI to engineer better.  I get super excited when I'm able to build something cool and hope to convey how much awesome stuff the world can build with this tech, but there is absolutely a balance with the harmful use cases as well.

If you made it to the end, thanks for reading!