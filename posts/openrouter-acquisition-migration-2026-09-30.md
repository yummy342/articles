# OpenRouter was acquired by Stripe. The migration checklist that actually matters.

Stripe agreed to acquire OpenRouter on 19 August 2026. The price was not disclosed; the press reported figures between $7 billion and $8 billion, mostly in stock. OpenRouter raised $113 million three months earlier at a reported $1.3 billion valuation.

Helicone, a different company in the same layer, also changed hands this year and is now described as being in maintenance mode.

Neither event changes your HTTP requests today. Both change what happens the next time you need something from the vendor. This post is the checklist I would run against any gateway before I let it into a codebase with a key in it. Every item here is one command, none of them need an account, and you can run them against mine at the end.

## The risk is not the acquisition

A gateway is two things at once: a company, and a shape. The company can be bought. The shape is what your code depends on, and it is the part that is expensive to get out of.

The shape is three decisions the vendor made before you arrived:

- which base URL your client points at
- what the model identifiers are called
- what the response fields mean

An acquisition does not change any of those. But it also does not protect them: the fastest way to make an acquired product cheaper to run is to change what it serves. Your client does not find out until it does.

## Five checks, five requests

Run these against whatever gateway you are on now. I picked the checks because each one fails *silently* in production.

**1. Does the documented base URL actually resolve?** OpenAI-compatible clients append their own path (`/chat/completions`), Anthropic clients append `/v1/messages`. So the base URL is only half a path, and a wrong one is a 404 that no API key can fix.

```
curl -s -o /dev/null -w '%{http_code}\n' -X POST https://<gateway>/v1/chat/completions \
  -H 'Content-Type: application/json' -d '{"model":"x","messages":[]}'
```

`401` means the route exists and you are missing a key. `404` means it does not exist. That difference is the entire test.

**2. Is the model list readable without a key, and does it carry a date?** If the catalogue needs a key, you cannot diff it against your own config without spending money. If a page quotes a count without saying when it was read, that number is already wrong.

**3. What are the model identifiers called?** This is the one that decides your exit cost. If the ids encode the vendor's supply chain (`somevendor/some-relay/model-name`), then every id in your code is a fact about their business, not about the model. When that supply chain changes, your strings change.

**4. Does the free tier require a card?** Test it with a fresh account, not by reading the pricing page.

**5. Does an over-tier request refuse, or bill quietly?** Ask for something you are not entitled to. You want a `402`, not a charge.

The fifth check has a second half that matters more than the first: ask what happens when a provider inside the gateway fails. An error returned to your client is honest. A silent fallback that swaps the model under you is not, and you will only notice it in the eval scores.

## Why the id check is the expensive one

I run a gateway, so I will use my own outage as the example. On 22 September our public model list returned 477 entries. On 29 September the same endpoint, unauthenticated, returned 199. Same URL, same response shape, different catalogue — because the implementation behind that URL had been swapped.

A count is the visible part. The damage was in the identifiers: the ids on the new endpoint carried the name of the upstream relay that served each model, which meant the old IDs resolved differently and one family of aliases stopped resolving at all. Any client that had cached the list, or hard-coded a model string, broke without an error message to explain it.

That is what an exit cost looks like before you are trying to exit. If the ids are neutral, moving is a base URL change. If they are not, moving is a migration.

## The test is portable. Run it on me.

Since I have a horse in this race, here is how the gateway I work on answers the five checks, and where it comes off badly.

```
1  base URL        401 on the chat route
2  model list      public, no key; 199 entries as of 30 September 2026
3  identifiers     neutral, fm/<model>; the old relay-prefixed names still resolve
4  free tier       no card; one tier is free to any account
5  over-tier       402, and the message names the tier
6  fallback        a direct model name is never substituted; the tier aliases are a
                   documented pool and will serve a different model from inside the tier
```

That last line is where we are deliberately weaker than the check implies. If you call a specific model by name, you get that model or an error. If you call a tier alias, you are asking for "any model in this band", and that is a substitution by design — it is documented, and it is the reason the free tier can survive a provider going down.

The honest column: there is no uptime guarantee and no third-party security certification. If your procurement process needs those, the absence is the answer, and a call will not change it. All six checks above are things you can verify in about ten minutes, which is the only reason I am comfortable printing them.

Run the six against every gateway on your shortlist, including this one. The results will tell you something the comparison posts cannot: which of them you could leave.
