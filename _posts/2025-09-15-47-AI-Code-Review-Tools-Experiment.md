---
layout: post
title:  "Experimenting with AI Code Review Tools at Playtomic"
date:   2025-09-15 09:00:00
categories: 
    - ai
    - engineering
    - code-review
permalink: /blog/:title
---

At Playtomic, we are constantly exploring how AI can improve our development lifecycle. After experimenting with different IDE tools, we decided to turn our attention to the Pull Request review process. 

For several weeks during the summer of 2025, we tested four different AI-powered code review tools: **[BugBot](https://cursor.com/bugbot) (from Cursor)**, **[Claude Code](https://claude.ai/code)**, **[GitHub Copilot](https://github.com/features/copilot)**, and **[CodeRabbit](https://coderabbit.ai/)**. Our goal was to find a tool that could provide meaningful feedback, reduce the burden on human reviewers, and maintain a high bar for code quality without breaking the bank.

## The Contenders

### BugBot (by Cursor)
BugBot was our first trial, activated across mobile, backend, and frontend projects.
- **Performance**: It reviewed 364 PRs and detected approximately 600 issues. Developers addressed about 45% of these issues. While many were cosmetic, developers noted that some were significant.
- **Pricing**: $40/month per PR creator (approx. $2,000/month for our team).
- **Verdict**: Very positive feedback on quality, but the pricing model felt too expensive at $40/month per user opening a PR. We stopped using the service until we could properly evaluate other market alternatives.

### Claude Code
Claude Code can run in headless mode and has a GitHub Action plugin to run on GitHub. With it, we can automate any task, including PR reviews.
- **Performance**: Very verbose output. While it found some potential performance and memory issues, it often missed architectural problems that human reviewers caught. It also struggled with "noise," reporting many trivial or false-positive issues. Comments are published in a single message, although inline messages is something that will be supported.
- **Pricing**: Token-based, roughly $0.50 per PR (approx. $250/month).
- **Verdict**: Cost-effective but high noise-to-signal ratio. In some tests, it was not capable of finding most of the issues related to architecture that a real developer found.

### GitHub Copilot
As part of the GitHub Copilot package, it includes a PR reviewer. It is fully integrated in GitHub, with nice inline comments and a PR summary.
- **Performance**: The feedback was generally less interesting than BugBot's. It excelled at PR summaries and minor cosmetic suggestions but lacked deep technical insight.
- **Pricing**: Copilot Business $19/month per user (approx. $1,000/month).
- **Verdict**: Great for summaries, but the issues reported are not very interesting for the most part, clearly behind the ones reported by BugBot.

### CodeRabbit
A specialized product that combines AI with static analysis tools like linters.
- **Performance**: Initial impressions were strong. It provided interesting diagrams for large PRs and caught subtle bugs, such as missing protocol implementations in example apps that would have caused compilation errors.
- **Pricing**: Lite for $15/month per user (approx. $750/month).
- **Verdict**: Good balance of features and price, with helpful visualizations.

---

## Head-to-Head: Claude Code vs. CodeRabbit

We ran several real-world PRs through both tools to compare their effectiveness. Here are some highlights:

### Case 1: UI for Organizer Sections
**Winner: 🐰 CodeRabbit**
While Claude Code provided more feedback, most were false positives (missing states or tests that weren't actually required for the scope). CodeRabbit's suggestions were more practical and accurate.

### Case 2: Bugfixes in Create Post Screen
**Winner: 🐰 CodeRabbit**
CodeRabbit correctly identified that some implementation files in example projects weren't updated, which would have broken those builds. Claude Code mostly suggested adding unit tests for existing logic.

### Case 3: Duplicate Items in Carousel
**Winner: ❌ Neither**
In several cases, neither tool found significant issues, often defaulting to minor nitpicks or suggestions that didn't apply to the specific changes.

---

## The Verdict: Killing the Experiment

After over two weeks of intensive testing, we reached a surprising conclusion: **the noise often outweighed the signal.**

By early September, our developers reported that they hadn't seen a single "game-changing" comment from the automated tools in weeks. Most feedback was either trivial, a false positive, or already caught by our existing CI/CD linters.

> "All PRs are either reporting nothing or giving false positives." — *Developer Feedback*

### Final Decisions

1.  **Removing Claude Code & CodeRabbit**: We have deactivated these integrations to avoid generating unnecessary noise in our PRs.
2.  **Keeping BugBot (Limited)**: We are keeping BugBot's free tier (3 reviews per developer/month) as it consistently provided the highest quality feedback. Developers can chose when to run those reviews.
3.  **GitHub Copilot**: Since it's bundled with our IDE tooling, we continue to use it for PR summaries, which remains its most valuable contribution to the workflow.

## Lessons Learned

Automated AI code reviews are promising but not yet a "silver bullet." For a team with high standards and existing robust linting, these tools can sometimes become a distraction rather than an aid. We will continue to monitor the market as these models evolve, but for now, the human eye (aided by BugBot's occasional intervention) remains our best line of defense.
