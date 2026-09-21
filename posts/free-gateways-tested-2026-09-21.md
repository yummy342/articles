# Free LLM gateways, tested: what the free tier actually gives you in September 2026

Every "free LLM API" roundup I can find has the same two problems. It quotes limits from the provider's marketing page instead of the API's own responses, and it treats the model count as a number you can compare across products. Both break within weeks.

So I checked the endpoints directly. Here is what six gateways actually give you this month, with the links to check each claim yourself.

## The free tiers, as of September 2026

| Gateway | Card required | Free allowance | Self-hostable |
|---|---|---|---|
| [OpenRouter](https://openrouter.ai/) | No | 50 requests/day; 1,000/day after $10 in lifetime credit | No |
| [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) | No | Analytics, caching, rate limiting and retries free on every plan | No |
| [Vercel AI Gateway](https://vercel.com/docs/ai-gateway/pricing) | No | A monthly free credit; the amount is not published | No |
| [LiteLLM](https://github.com/BerriAI/litellm) | N/A | The software is free | Yes, MIT |
| [Portkey](https://portkey.ai/) | No | 10,000 requests/month | Enterprise tier only |
| [FreeModel by Aiglade](https://freemodel.online/) | No | Free capacity is tried first, paid fallback behind it | No |

Every link above goes to the page that states the number, so you can verify or correct me.

**OpenRouter's free tier is tiered, and the number people quote is the wrong one.** The 50 requests/day applies until you have put $10 into the account, at which point it becomes 1,000/day. If you are evaluating it for something that might grow, the second number is the one that matters. The 5.5% fee on credit purchases, with a $0.80 minimum, is [documented in their FAQ](https://openrouter.ai/docs/faq).

**LiteLLM's "free" is free software.** You supply every provider key, plus the servers, the database and the maintenance.

## The model count is not a comparable number

This is the part that made me stop trusting roundups.

I run FreeModel, so I can audit my own numbers. On September 21 the public list at [freemodel.online/v1/models](https://freemodel.online/v1/models) returned 479 entries. Pull it again forty minutes later and it returned 494. The list is alive — providers add models, retire them, and change quotas on their own schedule.

And 494 is not one number either. Of those entries:

- **317** are text and multimodal models
- **177** are image-generation models
- **38** of the 317 are not models at all, but routing aliases like `auto/best-coding` that resolve to a different model at request time

So "494 models" and "317 models" are both defensible, and neither is useful on its own. A gateway listing 1,600 entries may be counting aliases, image models and deprecated rows in the same bucket as a competitor's 100. The count is only meaningful next to what it includes and when it was taken.

Any list you can check in one command is worth more than one you cannot. Here is ours:

```bash
curl -s https://freemodel.online/v1/models | grep -o '"id"' | wc -l
```

That is the whole audit. Ours answers without a key, which is the only reason I can write a number in this post and have you check it in ten seconds.

## What actually predicts whether a free tier works for you

**Whether the endpoint tells the truth about itself.** A public model list means you can verify the claim. An endpoint that demands authentication before it will tell you what it serves is an endpoint whose numbers you have to take on faith.

**What happens at the limit.** A free tier that stops answering and a free tier that silently starts billing you are different products. The second is how people find a $40 charge on a hobby project.

**Whether fallback is automatic.** Free quota runs out on the provider's schedule, not yours. If your code talks to one provider directly, every quota refill and every retired model is an outage you have to notice and fix by hand. A gateway turns that into a log line — but only if the fallback path exists and is on by default. [How ours decides](https://freemodel.online/routing/).

**Where the free quota comes from.** Almost every gateway's free capacity is provider promotional quota, passed through. The difference between gateways is what the router does when it runs out.

## Choosing by constraint

- **Traffic cannot leave your network** → [LiteLLM](https://github.com/BerriAI/litellm). Nothing else here runs inside your VPC.
- **You already live in Cloudflare** → [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/). The core features are genuinely free and the caching is useful.
- **You are on Vercel or Next.js** → [Vercel AI Gateway](https://vercel.com/docs/ai-gateway/pricing). Minimal setup, no markup on BYOK.
- **You have no provider accounts yet, or want free-first routing without writing the routing logic yourself** → [FreeModel](https://freemodel.online/). One key, one base URL, free capacity preferred, paid fallback behind it, and both OpenAI-compatible and Anthropic-compatible endpoints on the same address, so Claude Code points at it directly ([setup guide](https://freemodel.online/docs/claude-code/), [pricing](https://freemodel.online/pricing/), [key](https://freemodel.online/console/)).
- **You want the widest catalogue and do not mind a margin** → [OpenRouter](https://openrouter.ai/). Largest marketplace, though Stripe's acquisition of the company in August 2026 leaves future pricing an open question.

Full comparison against the alternatives, including where each is the wrong choice: [freemodel.online/compare/openrouter-alternatives](https://freemodel.online/compare/openrouter-alternatives/).

## Disclosure

I work on FreeModel, which is one of the six products above. Every number here comes from a public endpoint or a provider's own documentation — the links are in the table.
