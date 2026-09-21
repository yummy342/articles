# What AI engines actually cite when you ask about LLM gateways

We wrote 27 questions the way a developer asks them, sent them to four engines on one
day, and counted where the citations pointed. Here is the distribution — including the
findings that argue against our own product strategy.

## What we did

The 27 questions are the ones a developer actually types, not what a keyword tool
suggests:

- What are the best OpenRouter alternatives?
- Is it safe to route API calls through a third-party LLM gateway?
- Are free LLM APIs reliable enough for real work?
- How do I point Claude Code at a different API base URL?

Each question went to four engines on **2026-09-20**. Only one of them returns the
sources its answer was built from — Perplexity, reached through OpenRouter. An answer
with no citation trail cannot be measured, so the other three engines are not in this
dataset at all.

What is left: **613 citation entries** across the 27 answers, collapsing to **569 unique
URLs** and **346 unique domains**.

## 1. No single source dominates

The ten most-cited domains account for **18.6%** of all citations. The remaining 81%
spreads across 336 domains, and most of those appear exactly once.

This is worth stating plainly, because the standard advice in this space — get into the
big awesome list and you are done — does not survive contact with the data. There is no
list to win. There is a long tail, and each entry in it is a small bet.

## 2. The head still has a shape

| Domain | Citations | What the cited pages actually are |
|---|---|---|
| github.com | 17 | Awesome lists, tool repositories, and issue threads |
| dev.to | 17 | Long-form tutorials and comparisons, first person |
| openrouter.ai | 15 | A competitor's own product and pricing pages |
| llmgateway.io | 15 | Comparison blog posts, pricing, and legal pages |
| startupfortune.com | 11 | News coverage and tag archives |
| marktechpost.com | 10 | Category index pages |
| youtube.com | 9 | Tutorial videos |
| reddit.com | 3 | Discussion threads |

**The GitHub entries are not documentation.** They are *lists* — directories of free
APIs and gateways — and *issue threads*, where someone asks how to change the base URL
in Claude Code or Cline and other people answer. Both are places where a project can be
named without writing a blog post.

**The dev.to entries are all first-person and all contain numbers.** Not one of them is a
product announcement.

**And competitor marketing pages get cited directly** — including, for the question
about whether a gateway can read your prompts, a competitor's privacy policy and
sub-processor list. Those pages are usually written for lawyers. They are being read as
answers.

## 3. Reddit barely appears

Three citations out of 613 — **0.5%** — went to reddit.com.

That one deserves a pause, because "Reddit shows up in 40% of AI answers" is repeated
everywhere. The figure is real. It is also averaged across every topic imaginable. For
developer tooling questions, the engines reached for repositories and tutorials instead.
If you were about to spend this quarter on Reddit, the data says spend it elsewhere — for
this category, at least.

## 4. Where YouTube appears, it is someone else's tutorial

All nine YouTube citations are setup tutorials, and eight of them are about one
competitor, by name, walking through a configuration the viewer could copy.

## What we took away from it

**Publish where the engines already read.** For this category that is repositories,
issue threads, and long-form tutorial posts — not launch announcements.

**Your boring pages are answer pages.** Pricing, terms, privacy, and sub-processor lists
answer real questions that people ask engines. One competitor collected four citations
from legal pages in a single question.

**Expect the long tail.** 81% of citations spread across 336 domains. There is no single
placement that fixes visibility, which also means no single competitor can lock it up.

## Limits

One engine. One day. 27 questions, all of them ours, chosen to cover what we care about.
A different question set or a different date would produce a different distribution.
Three of the four engines returned no citations at all because they do not expose them —
so this study says nothing about how those three answer, only about which one is
measurable. Treat it as a snapshot, not a trend.

## Disclosure

We operate FreeModel, and our product is one of the things these 27 questions ask about.
In this round it received **zero citations**. Every competitor named above was cited more
often than we were.

We are publishing the method and the distribution rather than a conclusion, because the
number that matters is the one you can re-run. The full write-up, with the same numbers,
is on our site: https://freemodel.online/compare/ai-citation-study/
