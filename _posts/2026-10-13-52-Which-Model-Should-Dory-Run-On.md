---
layout: post
title:  "Which model should Dory run on? We tested eight on real questions"
date:   2026-10-13 10:00:00
categories:
    - ai
    - engineering
    - agents
permalink: /blog/:title
---

We put eight new AI models through the questions people actually ask Dory, Playtomic's internal AI agent, and had two AI judges from different companies check every answer against the code and the data.

*Figures as of 13 October 2026.*

## Meet Dory

Dory is Playtomic's AI agent for the whole company, and it lives in Slack. Engineers use it a lot, but most of the people asking it things work in customer care, product, marketing and other teams. Dory finds answers the way a colleague would: it queries Looker, reads code on GitHub, checks Linear, Notion and Datadog, and replies in the thread.

It handles around **1,000 conversations a month**, and a single answer can take dozens of steps. The questions fall into three groups:

- **Data** (about two in three): "How many clubs used X last month?"
- **Product behaviour**, answered by reading the code: "Why isn't this match showing up in the app?"
- **Knowledge and operations**: how a process works, who owns a flow, how to change a setting.

Today Dory runs on **GPT-5.6 Terra Pro**, from OpenAI.

## Why we ran this test

Almost every major AI lab has shipped a new model in the last few weeks, so we tested eight:

- **US labs**: OpenAI (Terra Pro, our current model, and GPT-6 Luna Pro), Anthropic (Claude Sonnet 5.5 and Claude Haiku 5.5) and Google (Gemini 3.8 Flash).
- **Chinese labs**, now very competitive on price: GLM 5.3 and DeepSeek V4 Pro.
- **Mistral Large 4**, Europe's flagship model, which interests us for that reason. It doesn't yet offer *Zero Data Retention* (ZDR), our requirement that the provider keeps nothing we send, but we tested it anyway.

### Why not just trust public leaderboards?

Public rankings measure generic skills: maths problems, self-contained coding, trivia. Dory has to read a hurried Slack question, decide where to look, chain many tools, make sense of Playtomic's own data and code, and reply briefly and correctly.

A model can top a leaderboard and still be mediocre at that. So we tested **the real questions Dory receives, answered by the real Dory**, changing only the model.

## How we ran the test

1. **Real questions, picked at random.** We drew a random sample of real conversations, keeping the usual mix of question types and removing personal details. People rate Dory's answers with a thumbs up or down; keeping only the well-rated ones would have favoured Terra, which we already know answers those well. **39 questions** made the final set.
2. **The real agent.** Every model ran inside the production Dory, with the same tools and instructions, and could only read, never change anything.
3. **Question by question.** Playtomic's data changes daily, so all the models answered each question back to back, against the same data.
4. **Rules written first.** We defined what "as good as the current model" means before seeing a single answer.
5. **Everything recorded.** The final answer, every intermediate step, the time taken and the exact cost.
6. **Two independent judges.** Two models from different companies graded every answer.

## The judges: deciding whether an answer is right

We used **AI models as judges**, but ours **check each answer instead of just reading it**. For every question, the judge first works out the right answer from the code, the documentation and the tickets. For data questions it checks each model's query, metrics and definitions, and whether the figures quoted match what the query returned. Then it grades each answer as correct, partial, wrong, unsupported or unanswered. Length, tone and confidence earn no points.

The two judges were **Claude Opus 5.5**, from Anthropic, and **GPT-5.6 Sol**, from OpenAI, each working without seeing the other's verdicts.

### How we tried to remove bias

- **Anonymous answers**, labelled A, B, C… in random order.
- **A second pass with the letters shuffled**, to check that position didn't affect the grade. The two passes largely agreed.
- **Two companies, averaged.** Each judge is slightly more generous to its own lab's models, and averaging both cancels most of that.
- **A control test.** A third judge, which compares answers in pairs, was secretly given identical pairs and called every one a tie.
- **Human spot checks** where the judges disagreed. The judge matched or improved on the manual check every time.

GPT-5.6 Sol is the stricter judge, but **both rank the models in the same order**.

## Results

We looked at three things, in the order they matter to Dory's users: is the answer right, how long does it take, and what does it cost.

### Quality: how often the answer is right

One point per correct answer and half a point per partial one, averaged across both judges, out of 100.

![Bar chart of quality scores: Claude Sonnet 5.5 74, Gemini 3.8 Flash 70, Mistral Large 4 70, GPT-6 Luna Pro 61, Claude Haiku 5.5 59, GPT-5.6 Terra Pro 56, GLM 5.3 56, DeepSeek V4 Pro 45](/images/posts/52/quality.png){:class="img-responsive"}

### Speed: in Slack, every minute counts

People wait for Dory's reply in the middle of their work, so a good answer in one minute beats a perfect one in fifteen. We measured the typical time and the slowest 10%, because slow answers are the ones people remember.

![Bar chart of answer times, typical and slowest 10%: Haiku 44 s and 1.4 min, Sonnet 51 s and 2.1 min, GLM 1.1 and 5.1 min, DeepSeek 1.3 and 4.0 min, Terra 1.9 and 4.4 min, Luna 2.3 and 6.7 min, Gemini 3.3 and 16 min, Mistral 3.5 and 17.5 min](/images/posts/52/speed.png){:class="img-responsive"}

Sonnet and Haiku are the fastest and the most predictable: neither ever took more than 3 minutes. Mistral and Gemini took over 10 minutes on about one question in four, and some came close to 25. That is the main reason Mistral isn't a candidate today, despite quality on par with the best.

### Cost: what each answer costs

Cost matters less than quality and speed, but at 1,000 conversations a month it adds up.

![Bar chart of cost per answer: Haiku $0.03, Luna $0.05, DeepSeek $0.23, GLM $0.27, Sonnet $0.35, Mistral $0.57, Terra $0.60, Gemini $0.79](/images/posts/52/cost.png){:class="img-responsive"}

**Price per token is not price per answer.** Gemini and Mistral are cheap per token but take so many steps, up to 38 per question on average, that each answer costs as much as our current model or more.

**Caching matters a lot.** It cut our costs to between a third and a sixth, and how much each model caches varies, so published per-token prices say little on their own. All costs here include caching.

### Putting it all together

![Scatter plot of quality score against cost per answer. Sonnet sits highest, Haiku and Luna are cheapest, and Terra is among the most expensive.](/images/posts/52/map.png){:class="img-responsive"}

| Model | Maker | Quality | Cost per answer | Typical time | Slowest 10% |
|---|---|---|---|---|---|
| **Claude Sonnet 5.5** | Anthropic (US) | **74** | $0.35 | 51 s | 2.1 min |
| Gemini 3.8 Flash | Google (US) | 70 | $0.79 | 3.3 min | 16 min |
| Mistral Large 4 *(no ZDR)* | Mistral (Europe) | 70 | $0.57 | 3.5 min | 17.5 min |
| GPT-6 Luna Pro | OpenAI (US) | 61 | $0.05 | 2.3 min | 6.7 min |
| **Claude Haiku 5.5** | Anthropic (US) | 59 | **$0.03** | **44 s** | 1.4 min |
| GPT-5.6 Terra Pro *(current)* | OpenAI (US) | 56 | $0.60 | 1.9 min | 4.4 min |
| GLM 5.3 | Zhipu (China) | 56 | $0.27 | 1.1 min | 5.1 min |
| DeepSeek V4 Pro | DeepSeek (China) | 45 | $0.23 | 1.3 min | 4.0 min |

## Where each model falls short

| Model | Weak spot |
|---|---|
| **Claude Sonnet 5.5** | Occasionally asks for clarification instead of answering. Otherwise the most accurate. |
| **Claude Haiku 5.5** | Gives partial answers to ambiguous questions. Weaker on code. |
| **GPT-5.6 Terra Pro** (current) | Asks for clarification when it doesn't know who is asking. Sometimes trusts outdated documentation over the code. |
| **GPT-6 Luna Pro** | Asked for clarification on over a third of questions when it didn't know who was asking. Some very slow answers. |
| **Gemini 3.8 Flash** | Gets stuck repeating steps; gave up on two questions after more than 200 steps. |
| **Mistral Large 4** | Extremely long investigations, up to 269 steps. No ZDR today. |
| **GLM 5.3** | Weak on code. Sometimes confuses who performed an action in the system. |
| **DeepSeek V4 Pro** | Claims things "don't exist" without checking and inflates figures. Attempted a write action nobody asked for. |

## Caveats

This was a practical test to choose a model for Dory, not a general benchmark:

- **One run per question.** Models don't answer the same way twice: re-running one on the same questions shifted its score by up to 10 points.
- **Only 39 questions.** Enough to trust the order of the models, not the exact gaps. Product (11) and process (2) questions are thinner still.
- **The judges are models too.** Averaging two companies reduces their self-preference but doesn't remove it.
- **Default settings**, including how much each model reasons before answering. Other settings could change quality and cost.
- **Mistral ran once**, with ZDR switched off for that test only.
- **A snapshot in time.** Providers update their models often; these figures are from October 2026.

## Conclusions

**Claude Sonnet 5.5.** First with both judges, typically answers in under a minute, and about 1.7 times cheaper per answer than our current model: around $350 a month.

**Claude Haiku 5.5.** About $25 a month for quality close to today's model, and the fastest of all. A good fit for simple tasks or as a second tier inside Dory.

**Mistral Large 4** has top-tier quality but is too slow for Slack and lacks ZDR; we'll test it again if both improve. **The Chinese models** aren't there yet for this use case, mostly on accuracy.

These results come from Playtomic's own questions, data, tools and priorities, such as fast answers in Slack. They may not hold the same way for another company or another use case.
