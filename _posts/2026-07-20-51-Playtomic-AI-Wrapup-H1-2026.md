---
layout: post
title:  "AI & Automation H1 2026 Wrap-Up"
date:   2026-07-20 10:00:00
categories:
    - ai
    - engineering
    - automation
permalink: /blog/:title
---

> This post is an adaptation of an internal wrap-up I shared at Playtomic to celebrate our AI journey in H1 2026. I've kept the public-facing industry insights and general learnings while removing internal-only details.

Happy mid-year! If you felt like AI was moving fast this half, you weren't imagining it. Frontier models got a serious upgrade, century-old math problems fell, robots ran half-marathons, data centers went underwater and into space... the world didn't slow down for a second.

Here's your skim-friendly wrap of what happened out there. 🌍

---

## 🌍 AI IN THE INDUSTRY

### 🧠 The Tech Race — Models & Capabilities

**Frontier models became genuinely dangerous.** The latest top-tier models can now find critical flaws in major software. Anthropic's answer was [**Project Glasswing**](https://www.anthropic.com/glasswing): keep the raw model locked, release a guarded public version — but [the UK government confirmed both approaches can be jailbroken](https://fortune.com/2026/07/10/openai-gpt-5-6-sol-jailbreaks-cyber-attacks-similar-to-security-flaw-that-led-u-s-government-to-force-anthropic-to-disable-fable-5/). Mythos stays locked, the jailbreaks and open models don't. 🔐

**Chinese models closed the gap — to weeks, not months.** [Kimi K3](https://venturebeat.com/technology/chinas-moonshot-ai-releases-kimi-k3-the-largest-open-source-model-ever-rivaling-top-u-s-systems) is the largest open-source model ever and placed ahead of Claude Opus 4.8 on real-world coding benchmarks, following DeepSeek R2 and Qwen 3 earlier in H1. US export controls were supposed to keep China behind. They didn't — the gap went from a semester to a quarter. 🇨🇳

**Anthropic overtook OpenAI — while the giants wobbled.** [Claude pushed past OpenAI in enterprise revenue](https://www.trendingtopics.eu/anthropic-overtakes-openai-in-revenue-hitting-30-billion-run-rate/), and key talent started crossing over — [Andrej Karpathy](https://techcrunch.com/2026/05/19/openai-co-founder-andrej-karpathy-joins-anthropics-pre-training-team/) among them. OpenAI's GPT-5 fanfare met the ["Goblin Incident"](https://openai.com/index/where-the-goblins-came-from/); [Gemini's next release slipped again](https://www.bloomberg.com/news/articles/2026-07-16/google-gemini-launch-delayed-as-tech-falls-short-of-internal-goals); [Meta quietly dropped open-source](https://thenewstack.io/meta-abandons-llama-spark/). Japan's answer: [**Fugu**](https://sakana.ai/fugu-release/), an orchestrator that beats every model by routing prompts across rivals. The winner wasn't who built the biggest. 🐡

**AI solved mathematics — not just one problem.** It's happening... it [cracked an 80-year-old Erdős conjecture](https://openai.com/index/model-disproves-discrete-geometry-conjecture/), [won gold at the International Mathematical Olympiad](https://www.scmp.com/tech/tech-trends/article/3319077/google-and-openais-ai-models-win-milestone-gold-international-mathematical-olympiad), and [proved the Cycle Double Cover Conjecture](https://aiweekly.co/alerts/openai-attributes-cycle-double-cover-proof-to-gpt-56-sol-ultra) in under an hour. This week, [AI disproved the 87-year-old Jacobian conjecture](https://thenextweb.com/news/jacobian-conjecture-disproved-ai-fable-5) — all while the World Cup final was playing. 🏆

**AI video learned to talk.** [Veo 3](https://felloai.com/2025/05/google-veo-3-the-dawn-of-ai-video-with-sound/) turned one prompt into video with synced sound, dialogue, and lip-sync. [Runway Gen-4](https://aicloud.press/en/tutorials/ai-video-generation-sora-runway) and [Kling 3.0](https://lensgo.ai/blog/veo-3-vs-sora-2-vs-kling-2-best-ai-video-model-2026) followed close behind — while OpenAI [quietly killed Sora](https://techxplore.com/news/2026-04-sora-shutdown-reveals-limits-ai.html). AI video clips flooded social feeds and most people didn't notice until reading the comments. If you're looking for the year AI movies truly began, historians will point to 2026. 🎬

**Personal agents went mainstream.** [OpenClaw](https://github.com/openclaw/openclaw) — a self-hosted personal AI agent — [broke every growth record in open-source history](https://thesoogroup.com/blog/openclaw-github-phenomenon-autonomous-agent-framework). What React took 10 years to build, it surpassed in 60 days. ⚡

> *My take: Models crossed a threshold this semester — smart enough to become a genuine security threat. Open models are closing the gap faster than anyone expected, and so far the speed of improvement shows no visible signs of slowing down.*

---

### 🏢 Enterprise Adoption

**Agents graduated.** Pilots to production: [31% of enterprises](https://gogloby.com/insights/ai-adoption-statistics/) now run at least one agent, and knowledge workers recover a [median 6,4 hours a week](https://www.digitalapplied.com/blog/ai-agent-productivity-statistics-2026-roi-data-points). But Gartner [projects 40%+ of agentic AI projects will be canceled by 2027](https://futurumgroup.com/press-release/enterprise-ai-roi-shifts-as-agentic-priorities-surge/) — unclear ROI, escalating costs, inadequate controls. Graduated doesn't mean easy. 🎾

**The AI cost paradox.** AI got roughly half as expensive — and somehow everyone's bill went up. The famous example: [Uber burned its entire 2026 AI budget in four months on Claude Code](https://fortune.com/2026/05/26/uber-coo-ai-spending-tokens-claude-code/) — token prices kept falling the whole time for similar capabilities, but usage exploded faster. When something gets cheaper, better, and easier all at once, "budget" becomes a different conversation. 📊

**The new challenge: self-improving AI.** Beyond one-off assistants, the industry is building systems that learn from their own wins and mistakes — getting more autonomous with every cycle.

> *My take: The companies winning with AI aren't the ones that spent the most — they're the ones that changed how they work.*

---

### 💰 Industry & Infrastructure

**The SpaceX arc.** [SpaceX IPO'd at $1,77T](https://www.cnbc.com/2026/06/03/spacex-ipo-stock-price-roadshow-musk.html), merged with xAI, then [acquired Cursor for $60B](https://techcrunch.com/2026/06/16/spacex-to-acquire-cursor-for-60b-in-stock-days-after-blockbuster-ipo/) — the largest startup deal ever, for a company four MIT classmates founded in 2022. Then the twist: [Anthropic pays xAI $1,25B/month](https://techcrunch.com/2026/05/20/anthropic-will-pay-xai-1-25-billion-per-month-for-compute/) to rent its GPUs. Your biggest rival might still be your best customer. 🚀

**Where to put the world's compute got creative.** China [built a data center 35 metres beneath the East China Sea](https://www.techradar.com/pro/china-unveils-worlds-first-underwater-data-center-2-000-server-facility-is-powered-by-offshore-wind-and-cooled-by-the-sea-making-it-one-of-the-most-efficient-around), cooled by seawater and powered by offshore wind. A [satellite now runs Google Gemma 3 in orbit](https://www.techtimes.com/articles/318563/20260617/satellite-ai-inference-clears-orbit-gemma-3-ran-aboard-yam-9-april.htm), with SpaceX planning full data centers in space. Meanwhile, [New York became the first US state to ban new hyperscale data centers](https://www.cnbc.com/2026/07/14/new-york-ai-data-center-ban.html) after electricity prices rose 58% in five years. Underwater, in orbit, and banned — all at once. 🌊🛰️

**The sustainability bill arrived.** Amazon's first-ever disclosure: [9,5 billion litres of water](https://theregister.com/on-prem/2026/07/10/ai-driven-datacenter-builds-drive-microsofts-emissions-up-a-quarter-in-one-year/5269924) in 2025 just for server cooling. Microsoft promised to go carbon negative by 2030 — and is now [building natural gas plants](https://techcrunch.com/2026/05/06/microsofts-ai-data-center-push-is-colliding-with-its-clean-power-goals/) just to keep the models running. 💧

> *My take: The investment is real, the infrastructure is real, and the demand is real. But so is the bubble — and the sustainability cost makes it harder to see how all of it stays alive at this scale.*

---

### 🌐 Politics & Regulation

**The Anthropic–government standoff: the defining story of H1.** Anthropic refused to drop two restrictions from its Pentagon contract (no mass surveillance, no autonomous weapons) — so Trump cut Claude from all federal agencies and [the Pentagon labelled Anthropic a **"supply chain risk to national security"**](https://www.technology.org/2026/03/06/pentagon-tags-anthropic-as-supply-chain-risk-over-ai-weapons-and-surveillance-fight/), the first American company to get a label. [Anthropic sued](https://www.cnn.com/2026/03/09/tech/anthropic-sues-pentagon). The Pentagon admitted it needed 6 months just to remove Claude from its own operations. A government that called a tool a national security risk couldn't live without it. 🏛️

**Four rival CEOs signed the same letter.** [Altman, Amodei, Hassabis, and Suleyman co-signed a Congress letter](https://fortune.com/2026/06/05/openai-anthropic-microsoft-ceos-congress-bioweapon-safeguards/): AI now outperforms PhD virologists on many technical questions, eroding the knowledge barriers that kept bioweapons inaccessible. When all four agree on something, take note. 🧬

**The UN created a global AI science panel.** [40 experts from all five UN regions](https://en.wikipedia.org/wiki/Independent_International_Scientific_Panel_on_AI), tasked with annual AI impact assessments. Their [first report, released July 1](https://www.manilatimes.net/2026/07/12/business/sunday-business-it/independent-un-panel-warns-ai-is-outpacing-the-worlds-ability-to-govern-it/2382705): AI is outpacing governments' ability to regulate it, human control is "not guaranteed", and AI is concentrating into a small number of hands — not becoming public infrastructure. The vote to create it: 117–2. One of the two against was the US. 🗳️

**The EU AI Act punted — but not entirely.** High-risk compliance [pushed to December 2027](https://www.traverssmith.com/knowledge/knowledge-container/eu-agrees-to-delay-key-ai-act-compliance-deadlines/). But transparency rules kick in August 2: label AI-generated content, tell users when they're talking to an AI. Fines when full enforcement arrives: [€35M or 7% of global turnover](https://www.legiscope.com/blog/eu-ai-act-timeline-deadlines.html).

> *My take: Governments are no longer asking whether AI should be regulated. They're trying to catch up with systems that already outpace their regulatory frameworks — and the gap keeps widening. But real regulation requires global coordination, and with China and the US moving in opposite directions, that seems further away than ever.*

---

### ⚖️ Safety & Ethics

**AI became a weapon — for fraud and for disinformation.** AI-generated images and videos flooded feeds at scale — from [synthetic missile strikes](https://www.profilenews.com/en/ai-in-global-elections-2026-explainer/) confirmed as real by Grok to [AI-made images of political opponents](https://www.aicerts.ai/news/how-political-misinformation-deepfakes-threaten-2026-elections/) distributed by government channels. A [single deepfake video call cost Arup $25,6M](https://memeburn.com/deepfake-statistics-2026-reveal-a-3892-fraud-surge/). The attacks don't need to be clever — just cheap enough to fire at scale. 💀

**AI joined the war room.** [US military personnel are giving chatbots target lists](https://www.technologyreview.com/2026/03/12/1134243/defense-official-military-use-ai-chatbots-targeting-decisions/) to help decide what to strike first. Now the robots are following: [a Trump-backed startup has already tested humanoid robots in Ukraine](https://www.cnbc.com/2026/05/30/humanoid-robots-ukraine-war-foundation-military-ai.html) and is preparing to give them lethal capabilities — "some kinetic things we're exploring," says the CEO. The Pentagon called "responsible AI" "utopian idealism" in a January memo.

**The resistance went mainstream.** [Hundreds marched past Google DeepMind, OpenAI, and Meta in London](https://www.technologyreview.com/2026/03/02/1133814/i-checked-out-londons-biggest-ever-anti-ai-protest/). [ChatGPT uninstalls surged 295%](https://www.euronews.com/next/2026/03/02/cancel-chatgpt-ai-boycott-surges-after-openai-pentagon-military-deal) after the Pentagon deal. MAGA Republicans, democratic socialists, and church leaders signed the same anti-AI declaration. Activists blocked or delayed [$130B in data center development](https://www.nbcnews.com/tech/tech-news/data-center-opposition-sharply-rising-2026-study-finds-rcna349728) in one quarter. Even [Pope Leo issued an encyclical](https://www.cbsnews.com/news/pope-leo-ai-encyclical-artificial-intelligence/) calling for the "disarming" of AI. The backlash is real, organised, and now has a papal bull. ✝️

**For the first time, "did AI make this?" has a technical answer.** A [coalition of Google, OpenAI, Adobe and 6.000+ others](https://c2paviewer.com/articles/openai-google-c2pa-synthid-2026) made content watermarking an industry standard — [20 billion images](https://internet-pros.com/blog/ai-content-provenance-watermarking-c2pa-2026/) and [1,3 billion TikTok videos](https://internet-pros.com/blog/ai-content-provenance-watermarking-c2pa-2026/) labeled so far. It's not perfect, but it's a start. ✅

> *My take: The attacks are real and scaling fast, and the industry is starting to react — but the pace of response still doesn't match the pace of the problem. Equally concerning is the combination of two things happening at once: open models becoming genuinely capable with no one controlling how they're used, and nation states already deploying AI in active warfare. That's a hard combination to walk back.*

---

### 🔬 AI Beyond Tech

**Health & Science**

**Three labs are racing to compress drug discovery by 10x.** AlphaFold cracked protein folding; now [Anthropic (Coefficient Bio)](https://pharmaphorum.com/news/ai-giant-anthropic-buys-coefficient-bio-400m), [OpenAI (GPT-Rosalind)](https://openai.com/index/introducing-gpt-rosalind/), and Google (AlphaFold 3) are competing to 10x the whole R&D pipeline. Protein-drug binding predictions run in seconds instead of hours, and researchers surfaced [**Manikomycin**](https://www.genengnews.com/topics/infectious-diseases/new-antibiotic-manikomycin-acts-on-novel-ribosomal-target/) — a new antibiotic hidden in a 75-year-old soil sample that kills drug-resistant superbugs via a mechanism never seen before. Drug discovery has never looked this exciting.

**Beyond drugs, a wave of milestones.** [CardiOmicScore](https://www.sciencedaily.com/releases/2026/07/260716023603.htm) can predict heart attack and stroke up to 15 years before symptoms — from a single blood test. [An age-reversal compound entered Phase 1 trials](https://gizmodo.com/first-human-receives-experimental-therapy-to-reverse-cellular-aging-2000769612). [A sesame-seed-sized '5-in-1' surgical robot](https://www.ntu.edu.sg/news/detail/a--5-in-1--seed-sized-surgical-robot) — cuts, biopsies, delivers drugs, generates cancer-killing heat — demonstrated by NTU Singapore. Not lab curiosities — clinical trials.

**And AI kept spreading everywhere.** [MIT found cement alternatives cutting CO₂ by 80–95%](https://news.mit.edu/2025/ai-stirs-recipe-for-concrete-0602). AI weather models beat physics-only forecasts on 97% of metrics — though they fail on events outside historical training data. H1 theme: AI finding things humans missed, in fields that had nothing to do with tech.

**Robotics**

**The humanoid moment arrived.** April 2026, Beijing: [a robot ran a half-marathon in 50:26](https://www.npr.org/2026/04/20/g-s1-118086/humanoid-robot-half-marathon) — 7 minutes faster than the human world record. 🏃 Production is scaling fast: [Tesla targeting 10M Optimus/year](https://www.therobotreport.com/from-evs-to-robotics-tesla-targets-10m-optimus-units-with-new-texas-plant/), [Figure AI from 1 robot/day to 1/hour](https://roboticsandautomationnews.com/2026/05/27/figure-ramps-humanoid-robot-manufacturing-at-unprecedented-speed/101954/), [Japan committing $6B to deploy 10M robots by 2040](https://www.webpronews.com/japans-6-billion-bet-10-million-ai-robots-to-fill-labor-gaps-by-2040). No longer a research project.

**Self-driving: fast and complicated.** [Waymo hit 500.000 paid rides/week](https://mezha.net/eng/bukvy/waymo_delivers_500-000/), fully driverless, across 10 US cities — the clearest proof yet that robotaxis work. [Tesla Robotaxi launched in Dallas and Houston](https://www.electrive.com/2026/04/20/tesla-launches-robotaxi-service-in-dallas-and-houston/), fully unsupervised. [China froze all new robotaxi permits](https://www.bloomberg.com/news/articles/2026-04-29/china-suspends-new-autonomous-driving-permits-after-baidu-outage) after 100+ Baidu vehicles simultaneously stalled in Wuhan, stranding passengers for hours. Three companies, three very different moments. 🚗

> *My take: The most surprising thing isn't that AI is helping with science — it's the speed. Fields that moved in decades are now moving in months. And the snowball is just starting to roll.*

---

## 🎾 AI AT PLAYTOMIC

At Playtomic, the direction is clear: make AI a practical part of how the company works, rather than a separate experiment. The focus is on giving teams better access to AI, turning shared knowledge into reusable workflows, and using agents to support people while keeping humans in control.

Across product, engineering, and operations, the goal is to reduce repetitive work, make knowledge and data easier to use, and help teams move from ideas to reliable outcomes faster. The next step is to make these systems more connected, capable, and self-improving—without compromising quality or safety.

Names behind this shift include Dory, Atlas, Tomi, Pulse, Rocket, and the AI Development Harness. At a very high level, they touch how teams find knowledge, understand the business, support customers, coordinate work, and build software. The important story isn't any single project, but the pace and breadth: AI is helping small teams turn ideas into useful systems across the company much faster than before.

---

## 🍿 IF YOU WANT MORE

**Movie: [Her](https://www.imdb.com/title/tt1798709/) (2013).** Before OpenClaw and personal AI agents went mainstream, Spike Jonze already imagined a world run by them. Fiction just caught up with reality — worth a (re)watch. 🎬

**Video: [This Paradox Splits Smart People 50/50](https://www.youtube.com/watch?v=Ol18JoeXlVI) (Veritasium, 26 min).** Not about AI — just pure fun. A mystery box, $1M and a super smart computer on the line. One box or two? I am a one boxer 🤣

---

*That's H1 2026. We covered a lot of ground — and so did the industry. 🎾*
