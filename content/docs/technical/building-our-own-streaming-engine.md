---
title: "Writing your own streaming engine is only a bad idea until it isn’t"
date: 2026-10-31
---

# Writing your own streaming engine is only a bad idea until it isn’t


*We run a [real-time data platform](https://aperiodic.io). After years of struggle we wrote our own streaming engine, surprise: with LLMs. We actually have two implementations, one in Rust and one in Go, and extensive parity tests between the two.
TLDR? Try creating bespoke implementations of databases, query engines, data pipelines from scratch, you may end up being surprised.*

For the last 9 months, we could go into depths that would felt impossible before.
We have a real-time data platform where ingest >1 million messages per second, we rebuild the order book from individual updates and we calculate useful and sometimes unique metrics. We process high cardinality time series, scaling out horizontally, with redundancy.

As you can imagine, “Just use postgres” doesn’t apply to us. We’ve spent years evaluating message brokers / streaming engines for latency, throughput, fault tolerance in a wide range of scenarios.
We have “harnesses” (excuse me, my vocab is not completely distorted by AI) that allows us to test new implementations for the individual angles we test for and all of them at the same time.

3-4 months ago, we thought - shall we, maybe try writing our own?
There’s this aura around “streaming engines” that’s they’re impossible to get right, but after a sleepless night patching a memory leak in the the fork of the open source streaming engine we used, we were at a breaking point and started a long-running session, not on the scale of “reimplement GTA 5”, but the data equivalent of that. Just to see what happens.
It always seemed like “these guys have spent a decade doing this, surely they’ve thought about everything”. We read their commits, listened to their podcast interviews, interacted with them.

Probably that’s true. But as we found out, we can now engineer performant code with LLMs tailored for exactly our use case that outperforms “stock” open source (yes, including QuestDB and kdb+) by a wide margin.
3 days later, it was on staging, in 2 weeks, it was in production.
We could resolve the edge cases we never thought we’d be able to resolve: we could add failure recovery behavior that we never thought anyone would cater for. We got to be 5x more efficient in cpu terms, and close to 10x reduction in memory.
We’re fully in control over the latency/throughput tradeoff.
We can extend the sql dialect with built-in functions that speed up the calculations of certain, unusual metrics very typical to our domain by 100x.

It wasn’t a single prompt, and there’s no secret sauce. We know exactly what our success criteria are and a lot about the edge cases (we’ve bumped into many) are.
We use software engineering best practices, and a very extensive unit/integration/e2e test suite with performance regression tests running on every commit.
Obviously, having a “ground truth”, the output of other query/streaming engines makes it easy to verify whether the reimplementation is correct. Although it’s useful to keep in mind that even those have errors/quirks that you may not want to re-implement.

On top of those, we do one thing that may worth a special mention:
We have two implementations in different languages (Rust & Go) running, compiling, tested and tested against each other in parallel, in the same repo. The parity tests that the agents are writing to compare these two implementations testing lots of edge cases, surfacing a lot of potential future issues upfront.
It eats tokens, but if correctness is a priority, we found them to be useful.
Go is always behind, as predicted (GC), although sometimes by not much.

Open source it? It would have been a dream a couple of years ago. Now, it feels like it’d b just an extra pain to maintain our private test suite on top of it, and deal with unsolicited PRs. We started on the work, but at one point, stopped.

I think you’re better off creating one for your own use case.

It’s only a couple of days more work than clicking “fork”, but it’s a lot more fun than that, I promise.
I’d just encourage everyone who’s inclined to do it to do it. The others who want an out-of-the-box open-source solution, I’m sure there will be plenty of those as well, by people who are better suited to maintaining it than us.

Our understanding of our streaming engine’s codebase varies a lot, depending on the angle. Some parts of it we know very much in detail, some parts of it were optimized to death to the point where we don’t follow. We know the acceptance criteria.

You want to speed up your query that’s currently running on DuckDB 2.0 by 2x?
I bet you real money any frontier model will get you there in a couple of hours, with parity tests. Of course it’s not gonna be full reimplementation, and it’ll only support the subset of sql functions you actually use, but extending it is as easy.

Do you want to optimize your BigQuery spend by a 100x by not running the compute on big query? I’d treat that as given.

I think the next wave of “data platforms” may be strict (!), use-case specific reimplementations of a small subset of query engines, compiled into a single binary, without a single redundant dependency. Although Apache Datafusion is pretty good, it won’t likely beat a custom, use-case-specific implementation that uses it as a reference. It’ll be Rust if it’s not overtaken by a similarly strict-but-faster-to-compile programming language.

I read what I wrote and completely emphatize with the “software engineering is now dumb” and “we’re completely just AI subordinates now” sentiment. But, what’s the alternative? Struggling with software that’s clearly not designed for our use case?
On our side, we clearly lack the lowest-level of software engineering skills, but can very effectively augment that gap now.

I’m not saying that we haven’t thought about writing a better compiler or unikernel for our usecase! Maybe that’s the next round of fun to be had.
