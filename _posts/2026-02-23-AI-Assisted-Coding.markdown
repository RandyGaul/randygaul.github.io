---
layout: post
title:  "AI Assisted Coding"
date:   2026-02-23
categories: coding
---

I recently had an AI implement a brand new lighting algorithm from a 2025 white paper, one-shot, producing visible soft-shadows and constant-time global illumination on the first try. Before that it one-shot an entire compute shader backend for my game engine. Tasks that used to take months took under an hour.

I'd been trying out AI for coding since ChatGPT dropped in 2022. Back then it was pretty trivial stuff -- converting enums from one format to another, scanning documentation for easy fixes, generating an initial documentation pass for a new function. Codex was a step up and could make pull requests straight to github. I used it to implement an OpenGL ES3 backend for my renderer, a pretty mechanical port just for emscripten. Codex actually one-shot an entire backend renderer by using my SDL_Gpu backend as a reference, and was able to mimic SDL_Gpu's *resource ring-buffer* API design for feature parity. Pretty cool. But it struggled with anything that wasn't so mechanical. Anything new, or anything requiring significant aesthetic/design oriented tradeoffs. I quickly gave up and went back to hand-coding.

With Claude Opus 4.5 I noticed something fundamentally changed. I'm not exactly sure what it was, maybe a few different things, like agentic technology becoming a little more mature, context window getting just large enough, and the LLM maturing enough. Who knows. All I know is after trying out a variety of AI tools this one has hit some kind of quality bar that unlocks some stunning results. You can take on significant coding tasks by describing context and auditing the AI implementation plans, and this more often than not takes much less time than manually editing text files.

# No Going Back

The genie really is out of the bottle on this one, and if you aren't using AI in some way for coding you're just falling behind. I get we all like different tools, have different styles, or whatever. Sure. Now I wasn't really alive in the 80s when C was gaining popularity, but from stories I've heard there was a lot of discussion in that decade about moving from assembly/fortran to C, a higher level language that automates a lot of the mechanical, bit-twiddling minutiae of managing CPU registers. And well, it won -- today we don't really hand-roll assembly except for odd cases, and most people don't even write in C anymore, opting for other higher level technology.

AI assisted coding is the next rung of that same ladder. The mechanical minutiae of managing functions, expressions, and statements gets taken over by a new abstraction. I still *can* write it all by hand, but I'd rather think in terms of context, design, and direction, and rely on AI to deal with wiring things together, mechanical refactors, boilerplate, etc.

There's a Pinocchio analogy here that I keep coming back to. Geppetto gets swallowed by the whale and survives by fishing inside its belly. He adapts just enough to not die, but he's stuck -- married to his crystallized knowledge of survival, unable to escape on his own. It takes Pinocchio, who is willing to try something genuinely new and unfamiliar, to actually get them out.

<div style="text-align: center;">
  <img src="/assets/whale.png" alt="whale">
</div>

If you're still not using AI for software engineering you're getting swallowed by a whale. Just kidding :)

# Examples

I had Claude implement compute shader support in [Cute Framework](https://github.com/RandyGaul/cute_framework) (the code powering my game project), something that had been frequently requested as a feature for years. This was a one-shot by Claude, of course with some pretty involved prompting, guidance, planning, etc. But still, what was taking years of procrastination to get done was finished in like... idk 30 minutes from inception to `git push`? Amazing.

The biggest single-task I've had Claude take on so far is to implement [Holographic Radiance Cascades](https://arxiv.org/pdf/2505.02041) (HRC), a new lighting algorithm published in late 2025. There's no way an AI could have trained on this paper prior and gotten released by the time I tried implementing, so it was all fairly new stuff, a pretty good test for the AI.

To help Claude out I actually took the pdf file of the white paper and had Claude help me read all the Greek, because well I don't speak white-paper, nor do I want to learn. Turns out all of the fancy equations are either function calls or for loops. Once converted to .txt and doing my best to help verify the transcription was accurate, I used the txt file version of the paper PDF as context for Claude to implement HRC. I basically had Claude go equation-by-equation and sketch out small structs or functions to build up small abstractions for the compute shaders.

After doing so it got the more complex merge functions, and I sort of just gave up following at that point to let Claude handle it. I figured if Claude could get out something that sort of worked I could tweak the shader code with the help of online shader hotloading, visually see the differences live, and work backwards to understand wtf was going on. And turns out this was a really time-efficient way to ramp up on understanding the algorithm and paper. In this way the AI was able to transcribe the language of white paper to the language of C and GLSL, languages I was actually familiar with already. This made the whole task of implementing the paper accessible to someone like me who is allergic to LaTeX.

The whole thing was coded about 90% by Claude, the last 10% being bugfixes, optimizations, customizations, etc. That first 90% was one-shot and *got something visible and coherent on the screen*. Here is the result after a Claude one-shot:

<div style="text-align: center;">
  <img src="/assets/hrc_one_shot.png" alt="one_shot">
</div>

Now obviously this screenshot is carefully cropped to hide all the bugs, but it certainly did produce some kind of soft-shadows, constant-time global illumination. And well, the results speak for themselves:

<blockquote class="twitter-tweet">
  <a href="https://twitter.com/RandyPGaul/status/2023576821904896113"></a>
</blockquote>

# Rethinking Code Maintenance Cost

There are plenty of other examples, such as solving bugs at work, accelerated code-search, cross-sectional design analysis of other popular games, the list goes on. Claude is *so damn good* at solving mechanical, task-oriented problems I started to realize I can shift my coding strategy to become way more effective in a weird way. I had a discussion with a friend of mine about hotloading assets in our game projects. I was describing how I had a pretty naive file-scanner for detecting if shader files are changed to hotload them while the game is running. This is a development-only feature, so who cares if it's pretty slow, right? My game doesn't have *that* many shaders, and SSDs today are pretty fast, right? I got away with this for a year or so until I started writing a lot more shaders. After some profiling I realized this was starting to take up like 1/3rd of my game's frame time.

I asked my friend about this situation, and they responded with what traditionally I would consider an extremely based take: "Just bind a keystroke like ctrl + R to reload all the shaders and do that each time you make an edit". Yes, this is an extremely simple, very robust way to do hotloading without affecting every single frame's performance. And usually I would agree, that anything more complex than this is just not worth the effort, specifically over the development of a game, which usually lasts years -- you absolutely can not afford extra things to spend energy on or split focus.

However, with AI today that's just not the same value proposition, and especially not the same risk any longer. AI is extremely good at implementing any technology that has somewhat aged, something with lots of old online documentation, blog posts, etc. So MSDN is basically free for the taking. I prompted Claude to write a file event watcher against the win32 API, something I really wouldn't ever tackle as a solo dev because it's just so far removed from the player experience. However, Claude can one-shot this super easily, and in the event of bugs popping up it's easy to prompt Claude about the bug and fix it in a matter of minutes. Which did end up happening, as later I noticed double-events were getting reported for some reason, and who cares what the root cause is, I just prompt Claude about it and it fixes the issue. This is only like 35 lines of code, dev-only, Windows only, so who cares. Honestly, it's just trivial with AI to deal with that kind of issue, an otherwise unholy concept pre-AI.

More generally, Claude is extremely good at writing small C programs to take on annoying tasks. You can just write a C program, stick it into your pre-build step in CMake, and rely on Claude to maintain that system over time as your dumb fucking CMake scripts magically decay for no reason, or you modify your project which cascades into little refactors for these C tools -- no problem. Claude can just do those refactors one at a time trivially, and it takes almost no time to do so. You can easily have AI setup regression unit tests as well, and have it run those itself, and even setup debug loops in the case of regressions that are almost nearly automated.

For example I went and downloaded some 2d tileset art packs as placeholder art for a game I'm working on. A common task. It's pretty time consuming though to use placeholder art made by someone outside of your immediate dev team because the art is never in the format you expect. Things need to be chopped up, shuffled around, and little tools are usually needed to facilitate the compatibility pipeline to get shit into the game. Use Claude for this. Describe the format verbally, explain specifically how to understand the engine format by merely pointing Claude to your source files, and just write a converter. This takes like 3 minutes, produces a standalone executable tool, and you can either check it into source or just throw it away. It doesn't even really matter, because producing a new one on-demand is still trivial, and will be trivial tomorrow, and will be trivial in 10 years.

I downloaded some Wang tilesets in dual-grid format, all to say they organized these 32x32 pixel tiles but the actual tiles I wanted to render were 16x16, and the ordering was all different compared to the custom ordering I had already defined pre-AI. I prompted Claude to write a C converter to chop up the image into 16x16 tiles, throw away tiles with no content (fully transparent tiles), and then reshuffle + keep the tiles I cared about that are compatible with my already defined in-engine format. One-shot, no problems. I simply checked in the converted tileset to git and moved on to the next task. This took maybe 5 minutes of my time.

All this to say, what once was too much burden to maintain becomes often unlockable with AI. The burden now moreso becomes the design of your architecture, the features you're implementing or planning, aka the actual project in your mind becomes the focus. This is much like scaling up your own little game studio and hiring employees -- you start delegating smaller scoped tasks, or tasks that contain less ambiguity and more specificity to others, freeing up your time to focus on something else.

# AI Coding Flow

Let's compare traditional and AI assisted coding flows:

```
              TRADITIONAL HAND-CODING
    ┌─────────────────────────────────────────┐
    │                                         │
    │  Understand ──► Design ──► Write Code   │
    │   Problem       Solution   Line-by-Line │
    │                                 │       │
    │                      ┌──────────┘       │
    │                      ▼                  │
    │                 Debug & Test            │
    │                      │                  │
    │               ┌──────┴──────┐           │
    │               ▼             │           │
    │           Ship It       bugs│           │
    │               │             │           │
    │               ▼             │           │
    │     ┌───────────────────┐   │           │
    │     │  Maintain Forever │───┘           │
    │     │  (refactors,      │               │
    │     │   bit-rot, CMake  │               │
    │     │   decay, compat)  │               │
    │     └─────────┬─────────┘               │
    │               ▼                         │
    │   ┌───────────────────────┐             │
    │   │ "Is this feature      │             │
    │   │  worth building AND   │             │
    │   │  maintaining?"        │             │
    │   │                       │             │
    │   │  Often: NO ──► not    │             │
    │   │           built       │             │
    │   └───────────────────────┘             │
    │                                         │
    └─────────────────────────────────────────┘
```

The really expensive bits are when you're stuck in a loop of debug & test/bugs and have to repeatedly go back to design/write by hand. The by-hand or line-by-line part is very time consuming and honestly is a lot of hard work. At the end of the day, in order to really maximize your own time efficiency you have to be really brutal about tradeoffs. Code starts to look *extremely expensive*, less code is better, ease of maintenance is priority, lowering risk over time is priority. These are all still true even with AI coding, but... Well just look:

```
               AI-ASSISTED CODING
    ┌─────────────────────────────────────────┐
    │                                         │
    │   Describe Context ──► AI Plans &       │
    │   & Direction          Generates Code   │
    │        ▲                    │           │
    │        │                    ▼           │
    │        │            ┌──────────────┐    │
    │        │            │ Review &     │    │
    │        └────────────│ Audit Output │    │
    │        refine       └──────┬───────┘    │
    │        prompt              │            │
    │                     ┌──────┴──────┐     │
    │                     ▼             │     │
    │                  Ship It     regen│     │
    │                     │        from │     │
    │                     ▼        desc.│     │
    │           ┌──────────────────┐    │     │
    │           │ Maintain?        │    │     │
    │           │ Or just regen    │────┘     │
    │           │ on demand.       │          │
    │           │ Cost ≈ trivial.  │          │
    │           └─────────┬────────┘          │
    │                     ▼                   │
    │       ┌───────────────────────┐         │
    │       │ "Is this feature      │         │
    │       │  worth describing     │         │
    │       │  clearly?"            │         │
    │       │                       │         │
    │       │  Usually: YES ──►     │         │
    │       │       gets built      │         │
    │       └───────────────────────┘         │
    └─────────────────────────────────────────┘
```

You just have this undying minion who never tires, always available to plow through the dirty work. It's like growing 10 arms and 3 brains or something. Time efficiency just goes way up because iteration time between specifying the problem and shipping a thing gets just absolutely smashed by AI. Here's another visual for you, rare footage of Claude smashing bugs:

<div style="text-align: center;">
  <video src="/assets/myjzZzDHltHD.mp4" autoplay loop muted playsinline></video>
</div>

When faced with problems, bugs, refactors, etc. it's just usually pretty trivial to generate prompts to investigate further, debug, refactor, or try different strategies. Typically once a problem is described with clarity there's a cascade of simplicity and clarity that follows through the implementation. Bugginess, instability, or otherwise code issues usually stem from inherent design or approach flaws. By focusing on the clarity of "documentation" the problem transforms from many verbose but mechanically simple operations, to a higher level form where we more just talk like a humans talk to each other.

All this to say, problem solving gets lifted up into a new paradigm. You can work more as a code or problem designer, and think more in terms of product or implementation strategy. The cost of trying out a refactor is now often lower than actual penciling out every detail beforehand. Often I find myself just telling Claude to try out an idea or specification before I fully understand all the implications, because I realized I can just open my game project and try it out empirically. If it looks promising, *then* I can invest precious human time to review and think through implications -- often prompting followup reimplementations or designs, but *more informed* reimplementations or designs.

# Conclusion

For me the conclusion is solid here: I've been able to smash dozens of open source issues/bugs/tickets that would otherwise have taken years to get through in my spare time, all in one weekend. Tasks that would normally take 1-2 months to ship (implement, test for bugs, verify robustness) took usually under an hour. Given a well specified problem Claude can just churn through tasks, wire things up, and otherwise interpolate between the rest of your code as reference points. I would be a little wary setting up an entirely new project purely with Claude, but for working in a pre-existing codebase with a lot of prior pattern references Claude can nail about 95% of prompts I throw at it first try, writing code *very close to what I would otherwise have written myself by hand*.

And to me that last part is crucial: I really feel like it's been able to get extremely close to code I would actually have written by-hand in practice. I do wish corporate environments I work in would catch up in terms of policy and security, because at least for my own projects I've saved months and months of time, and actually have gotten to a point where I've *run out of coding tasks* and have to start hard-pivoting into design and content creation mode to *generate more coding tasks* for AI to smash through. I've never in my life been in this position. The bottleneck has always been implementation speed and maintenance cost. It's just such a shock to have that bottleneck shift to some other domain.

The advent of AI isn't some passing fad, it's a fundamental shift in daily life and will pervade most aspects of life, kind of like the advent of the internet. Or in our AI coding case, kind of like the advent of compilers to produce assembly. Don't be Geppetto.

# Some Parting AI Tips

Claude's bottleneck is, in my opinion, memory of the context window. It's only 100k tokens, whatever that means, and it ain't much. I've found a trick to help here is write a claude.md file for your project that points to files to reference. At first I tried writing actual documentation briefs in claude.md, and found it would read those and then just voluntarily go read the source files anyways for more of a concrete look. Which you know, fair enough. I'd do the same, personally! So in the end just routing AI straight to the relevant source files saves a lot of parallel-scan sub-agents from spawning.

However, that parallel-scan of sub-agents is going to happen one way or another. The AI needs its precious context, and who are you to deny Gollum his precious? Don't deny Gollum of his precious. Feed the little mouse its cookie and soak up 20-30% of your context window as the session begins. Once you approach that devastating 10% until the auto-compact, your life isn't actually over. Just use the `/rewind` feature and select `conversation only` to keep your code changes. Back up just to the point where Claude did its initial breadth-context-scan and keep that. Reuse your session to solve the next problem, next iteration, etc. and avoid that apocalyptic conversation compaction.

Another really nice couple of features are the `/plan` mode, really my favorite way to make sure to stave off Claude's retardation. And yes, it does have like 95% correctness in most cases, as in, 95% of the code or plan is what I would have done myself by-hand, but that 5% is real and does compound. If I let in 4 tasks at 95% correctness that will compound down to 81% correctness, and it does continue decaying. Human vigilance is required. `/plan` is great for that. Another way to help with plans or task-specificity is to tell Claude to drill down on ambiguity with the AskUserQuestion tool. It's also just fun to use the tool because it asks you multiple choice questions. And often asks good questions, like the kind it would have otherwise taken unspoken assumption on (often that 5% of retardation comes from weird assumptions Claude gets married to in the dark).

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>