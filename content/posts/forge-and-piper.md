---
title: "Forge and Piper"
date: 2026-04-03
draft: false
description: "Building my own harness - Forge and testing it out by building Piper - a simple ios app that helps you pipe auth-walled articles to any read-it-later app"
tags: ["ai", "agents", "harness", "claude code"]
links:
  forge: "https://github.com/m6un/forge"
  piper: "https://github.com/m6un/piper"
---
So I read this article from OpenAI on Harness engineering[^1] on a flight from Calicut to Bangalore and ever since that I've been working on building an ever improving harness for my side projects. The harness, which I named **Forge**, would allow a constrained environment for AI agents to build, verify, publish, and review their own code.  
 To test this out, I built **Piper**, a simple ios companion app that helps you pipe auth-walled X articles into a read-it-later app of your choice.
<img src="/images/harness-diagram.png" alt="harness diagram" caption="Piper harness diagram" style="margin-left: 0">

### Building piper with the harness
Here's what we built:  
The idea is simple - you log into X inside the app, paste an article link, and Piper extracts the full content behind the
  paywall, saves it to a temporary backend, and copies a shareable URL to your clipboard. Send that to Instapaper Pocket, whatever you use.
- Backend: A Cloudflare worker with accepts a POST /save endpoint, GET/{uuid} endpoint and a KV storage with 1hr TTL
- Ios: A content extraction pipeline - X login via *WKWebView*, content extraction (desktop UA, SPA-aware retry), backend save and automatically copy the UUID URL to clipboard once copied

I started with having a proper doc setup[^2] and included everything from design docs to product specs to quality and reliablity metrics. It's meant to be an exhaustive list of guides for the agent. I kept my CLAUDE.MD file very minimal ( ~50 lines) and included a table of contents inside it which the agent can refer according to tasks that it's working on.  
We added the loops next. This includes:  
1. A **/build** loop -> spec → worktree → builder agent → verify → publish PR → reviewer agent → pass/fail. Max 3 cycles, then escalation
2. A **/fix-review** loop -> reads human PR comments → fixes in worktree → verify → push
3. A **/bug-fix** loop -> bug report → worktree → diagnose → fix → verify → publish → review

Scripts were a part of these loops which helped with worktree creation, output verification using custom linters and invariants and publish / escalate a PR.  
A **retro agent** was one of the most interesting additions to the setup. Retro agent runs ater every build. It reads the spec, review findings, breadcrumbs left by the builder agent while going through build cycles, diffs, agent prompts and prior retros. It classifies root causes into spec gap, builder or reviewer misses, linter or script or infra bug and raises a PR of it's own with the analysis and the fix. Interestingly, retro agents made 8 self-improvements to the harness.
### Forge as a CLI
I was able to get the ios app working without writing a single line of code. This proved that the harness works. I went ahead and bundled it into a CLI tool. Think create-react-app but your coding agent can use the cli to create a custom harness for your side project idea and once done, all you have to do is write clean specs and yolo with the build loop.

[^1]: https://openai.com/index/harness-engineering/
[^2]: https://github.com/m6un/piper/tree/main/docs
