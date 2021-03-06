---
layout: post
title: How to peg your pipeline to a dependency version
status: public
type: post
published: true
author: Sriram Narayan
---

Say we have a set up like the one below. We have two pipelines -- one for component-1 (C1) and another for component-2 (C2). C1 just builds off its source code in version control (VCS-1). C2 has its source in a different repository (VCS-2) and is also dependent (d3) on C1. In Go terminology, C1 has one upstream dependency d1 while C2 has two upstream dependencies d2 and d3.

![](/images/blog/sriram-peg1_0.png)


###Fluid dependencies

Now by default in Go, a pipeline gets scheduled when there is a change to any of its upstream dependencies. This behavior is referred to as a *fluid dependency* in the [Continuous delivery](http://continuousdelivery.com/) book (the book refers to a paper by Alex Chaffee).

>Chaffee’s proposal is to introduce a new piece of state into the dependency graph—whether a particular upstream dependency is static, guarded, or ﬂuid. Changes in a static upstream dependency do not trigger a new build.Changes in a ﬂuid upstream dependency always trigger a new build.


###Simulating static dependencies

What if we want to keep d1 and d2 fluid but make d3 static? This is a reasonable requirement if you want to peg C2 to a known good version of C1 and still let C1 keep building against its frequently changing source. This may be the case if C1 is volatile or under control of a different team or organization.

On the face of it, it may appear that Go cannot support pegging. But that is not the case; Go’s pipelines are powerful and flexible building blocks. We could just introduce a “manual-gate” pipeline to achieve pegging.

![](/images/blog/sriram-peg2.png)


“manual-gate” is a simple pipeline with one stage having a single no-op job, one upstream dependency C1, and [auto-scheduling turned off](http://www.thoughtworks-studios.com/docs/go/current/help/configuration_reference.html#approval) (this makes d3 static rather than fluid). The manual gate doesn’t trigger for every successful run of C1, it can only be manually triggered. C2 has a regular fluid dependency (d4) with manual-gate. The overall setup pegs C2 to a chosen good version of C1 while C1 is free to keep building new versions.


###Rolling back

Say C2 is pegged to version 100 of C1. A week passes and C1 has progressed to 110. To get in sync with latest again, we simply trigger the manual-gate. The manual-gate itself is a no-op so it turns green and triggers C2. C2 does a [fetch-ancestor-artifact](http://www.thoughtworks-studios.com/docs/go/current/help/managing_dependencies.html#fetch_artifact_section) to get C1 binary version 110 and proceeds to build. If the build turns green, it means we have successfully pegged C2 to version 110 of C1. What if the build fails?

We have two options. We could revert to last known good version by [retriggering manual gate with version](http://www.thoughtworks-studios.com/docs/go/current/help/trigger_with_options.html) 100 of C1. Or we could keep bisecting the range between 100 and 110 to find the most recent version of C1 that works for C2.


<div class="highlight">This post is also cross-posted <a href="http://www.thoughtworks.com/insights/blog/how-peg-your-pipeline-dependency-version">here</a>.</div>
