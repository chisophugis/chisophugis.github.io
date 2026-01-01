# First Thoughts on AI Coding

I've spent the past week building a project from scratch using agentic coding tools—a few hours a day across Claude Code, Codex CLI, Google Gemini/Antigravity, and Cursor.

Some patterns are already emerging. These tools seem to have a surprisingly specific profile of strengths and weaknesses, and the way you work with them matters a lot.

## My Tentative Mental Model

If there's an underlying principle across these approaches, it might be: **match the level of AI autonomy to your ability to specify the task.**

- For grinding, verifiable work → let it run.
- For architectural decisions → stay in the loop.
- For taste-sensitive code → build incrementally and feel the pushback.

The corollary to this is that you should optimize your codebase design to maximize agent autonomy. More autonomy = more work done per unit of *your* time.

## Where AI Seems Strong (and Where It Doesn't)

### Seems Genuinely Strong

**Verifiable grind work.** Anything with a clear success criterion that requires tedious iteration—debugging, profiling, performance optimization, making tests pass—seems to be a sweet spot. The agent can hack up the code, check results, and iterate without me in the loop. Some agents can even do browser-based debugging.

**Reading and navigating large codebases.** AI appears to ingest and reason about far more code, far faster, and at a far greater level of detail than I can. I find it's just not worth it to try to trace through code across the codebase when I can instead ask the agent.

**Writing code extremely fast.** AI can write implementation code so fast that it really changes the equation for how to work. Implementation code can almost be considered throwaway / "rewrite on-demand" as long as the interface and testing are good.

**Working in the background.** AI works in the background, so even if it takes 5 minutes on something, if there's something else I can do, it's "free".

**"Just knowing stuff."** Libraries, idioms, file formats, API conventions—the encyclopedic knowledge is real. It's able to mix and match libraries in a way that would require a lot of research for me as a person. Once the code is written, it's much easier to reason about and poke at to fill any pressing knowledge gaps.

**Boilerplate and glue code.** Extracting a formal JSON schema from a plaintext description, wiring up API endpoints, writing serialization code—anything repetitive and mechanical seems to go smoothly. Almost like Don't Repeat Yourself is less of an issue, as long as there is a way for the agent to propagate mostly-mechanical changes thoughout the codebase.

**Catching "dumb mistakes."** AI rarely seems to forget to update tests when changing a function, or leave a dangling reference when renaming something. I find I lean on the AI for even basic edits just to avoid the occasional typo or forgotten thing.

**UI decisions in web development.** Surprisingly, the agents make very tasteful choices for layout, colors, spacing, and visual hierarchy. I didn't expect this considering they are editing code purely in text.

### Seems Okay with Guidance

**High-level architecture.** AI can brainstorm options, but I've needed to drive these conversations. It doesn't instinctively know which tradeoffs matter (even when a good human software engineer with no context would).

**System boundary design.** What data structures cross boundaries? Where should the seams be? AI can help explore, but I've found I need to validate carefully.

### Where I've Seen It Struggle

**Software engineering "taste."** At the individual module level, good layering, clean abstractions, appropriate boundaries—the agents don't "feel the code pushing back" and start hacking things together. That's fine if you can limit the blast radius of those decisions under a good interface though.

**Type discipline.** Especially in Python and TypeScript, I've seen the agents default to loose types—raw dicts, strings, no enforced invariants. The code works but feels fragile. (I'm curious how much statically typed languages would help here.)

## Techniques That Seem Promising

Here are some techniques / patterns that have worked for me so far. Your mileage may vary.

### 1. "Throw One Away"

When AI-generated code doesn't work and debugging isn't converging, I've started just deleting it and trying again—often with a smaller goal that isolates what didn't seem to be working (e.g. "WebGPU couldn't render onto the canvas -- let's first see if I can draw on a regular 2D canvas").

This felt wasteful at first, but it's often been faster than debugging a fundamentally (or subtly) flawed approach. The AI is just so damn fast at writing code (and it does it in the background) that the economics actually work.

### 2. Build Incrementally

With this technique, I basically treat AI as an "augmented keyboard"—working in small class/function-sized increments rather than asking for entire modules.

This approach seems to let me notice when something feels wrong—when the abstractions are getting awkward or the layering isn't clean. I can course-correct before I'm deep in the weeds. Someone described this as feeling the code "push back," and that resonates.

**What I've been doing:** Making WIP commits after each piece of working code, then squashing at milestones. This gives me checkpoints to roll back to in case the AI makes mistakes.

**The tradeoff:** This is slower (has me more in the loop) than letting the agent run autonomously. But for code where I care about the architecture, it seems worth it.

### 3. Run Multiple Agents in Parallel

A few times I've given the same prompt to multiple agent sessions, compared their approaches, and cherry-picked the best ideas from each.

This seems especially useful for ameliorating the AI's poor software engineering taste. It's easy for me to scan multiple implementations and see which one did things better.

**The tradeoff:** More tokens, more time reviewing.

### 4. Make Everything Verifiable

The more I give the agent access to—tests, linting, type checking, formatting, browser automation—the more autonomously it can iterate. A tight feedback loop (write code, run tests, see failure, fix) is something AI seems to handle well.

I've been trying to maximize the "verifiable surface area" of my workflow. If the agent can verify its own work, it needs less babysitting.

**One thing I've noticed:** If setting up integrations feels burdensome, the AI is pretty good at helping with the configuration work itself.

### 5. The "Interview" Approach for Specs

Instead of writing a spec myself, I've tried having the AI interview me about the system based on a loose file of random bullet points I've made. It asks questions; I answer. The prompt includes something like:

```
The goal is to flesh out:

- What are the components of the system?
- What are the interfaces between components?
- What are the "day in the life" flows through the system?

The output should be concrete: function signatures, struct definitions, database schemas, file paths. 
```

Then do the same for milestones and testing strategy.

**The tradeoff:** The interview takes real time—more than I expected. But the result is pretty good.

### 6. Have the Agent Build Its Own Context

Using the agent to help guide future agents. Examples:

- "Update agents.md with what you learned in this conversation"
- "Take what you learned in this conversation and create a development guide for this component"

The idea is to amortize effort—invest once in good context, and every future session benefits.

**One caution:** The agents tend to over-document. I've had to pare down what they produce.


## Looking Forward to the Future

I think a lot of these will get better naturally over time


**AI is getting faster/cheaper *really* fast.** A lot of things could be handled by the AI themselves, but would cost more tokens (either too expensive or too slow). Thankfully, cost per token is decreasing ~100x per year, and speed (tokens per second) is increasing by ~10x per year.

**AI is getting more connectors.** The AI will naturally get more connectors (GitHub/browser/etc.) integrated so that they can be more autonomous. Each new tool can be mixed and matched with other tools by the agent, giving a compounding autonomy to the agent.

**Building taste into the AI's soul.** I suspect the AI labs will train in more software engineering "taste" and "common sense" into the weights of the model. In my experience, AGENTS.md doesn't do too much for this.

**Larger/older codebases.** I have yet to see how this plays out in larger/older codebases. I suspect that we will need to architect code differently -- much smaller modules, much more focus on interfaces, tests, and verifiable feedback loops as the "artifact" that a software engineer puts thought into.

**Takewaway.** I think these tools are here to stay, and we aren't even scratching the surface of what they are capable of.