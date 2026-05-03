---
title:  "Re-imagining Github/Gitlab Actions"
date:   2026-05-02
---

# Re-imagining Github/Gitlab Actions

As with most of you, our small team’s velocity has increased a lot over the last 6 months.
There is a “reimagining git(hub)” zeitgeist going on, and “actions” are a big part of that in terms of time and money spent for every productive team. They’re even more important now, right?

I think a world where llms/agents write most of the code (they do on our side), there are a couple of tweaks that may be optimal for “actions”:

- Fine grained permissions of what kind of actions or steps could be triggered by what kind of agents.

  A “developer” agent working on a feature branch shouldn’t have deploy access to any of the environments. Of course this assumes that there are agent roles already, that are enforced across the different clients (or when communicating via the GH/GL api).

- Checkpoints / Failure callbacks

  Loop back to the agent that written the code immediately after a step failed, and give them the ability to observe the point-in-time state, propose a patch and re-run the step.
  Versus destroying the current container and state and re-starting from scratch. (While a dry-run is necessary for final validation).

- Optional, dynamically injected step

   Calling an agent for a quick sanity check whether the intended behavior did happen - a more qualitative version of a “rigid” acceptance test, acting as a sanity check. We all have seen llm written tests that test the wrong thing. Arguably, that happens uncomfortably frequently!

- Streaming logs / observability

  I mean this doesn’t feel like a huge ask, the api doesn’t allow streaming the logs but the UI does.

- Human-in-the-loop approval gates
  
  So you can’t wipe your production cluster accidentally, even if you have a GitHub action for that.
  More realistically, so Claude doesn’t end up “I couldn’t deploy to try this to a dev environment so I’ll just repurpose the production environment to test this experimental feature.”

- Maybe the whole graph could be made more dynamic

   Have an agent upfront determine “Given this PR diff, repo history, dependency graph, flaky-test history, and previous failures, here is the CI plan I will execute.” Yes there’s the `changes` syntax but we all know it’s too brittle.

I assume everyone sees that agents are more helpful if you use them end-to-end, and the current Github/Gitlab/Gitaea actions are a bit in the way.

These ideas assume that you’re running the full suite tests on every commit. 
I assume you still want to do that before merging a branch!

