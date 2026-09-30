# I read the public model list of fourteen gateways. Five of them publish one.

Ask a gateway what it serves and you get one of two answers: a list, or a login form. I wanted to know which is more common, so I called the documented endpoint of fourteen gateways — no account, no key, one GET each.

Five answered with a model list. Eight answered `401` and meant it: the catalogue is behind an account. One answered only after authentication. Two I could not reach from where I ran this at all.

None of that is a scandal. But it changes what a migration costs, and it is the thing I would check before picking a gateway rather than after.

## What I ran

For each gateway, the documented OpenAI-compatible base URL, with an empty body and no credentials:

```
curl -s -o /dev/null -w '%{http_code} ' -X POST https://<gateway>/v1/chat/completions \
  -H 'Content-Type: application/json' -d '{}'
curl -s -o /dev/null -w '%{http_code}\n' https://<gateway>/v1/models
```

`401` or `400` on the first call means the route exists and is waiting for a key. `404` means no key will ever help — the documented base URL is wrong. On the second call, `200` means you can read the catalogue before you sign up.

Run on 30 September 2026. Status codes as returned that day.

## What came back


Every reachable gateway's chat route answered with an authentication error rather than a `404`. That is the good news, and it is worth stating plainly: I did not find a single documented base URL that was simply wrong. The failures people hit in this category are elsewhere.

```
OpenRouter            chat 401   models 200   464 entries
Requesty              chat 401   models 200   749
Vercel AI Gateway     chat 400   models 200   395
LLM7.io               chat 400   models 200    65
FreeModel             chat 401   models 200   199
Groq / Together / DeepSeek / SiliconFlow / Helicone / Portkey / Z.ai
                      chat 401   models 401
Mistral, Gemini (OpenAI compat)   unreachable from this network
```

## Why the list matters more than it looks

A public catalogue is not a marketing page. It is the only way to answer three questions without paying:

1. **What did you serve last month, and what do you serve now?** You can diff two reads. If the list needs a key, you can only see the current state, and only after signing up.
2. **What shape is it in?** Field sets differ more than I expected — six fields on one gateway, thirty-four on another, eighteen on the model labs. Anything your tooling derives from those fields is a bet on one vendor's schema.
3. **What do the identifiers encode?** This is the one that decides your exit cost, and I will come back to it.

Of the five that answered, the field sets share almost nothing beyond `id`, `object` and `created`. One publishes per-model pricing and retention notes; another publishes a knowledge cutoff and a Hugging Face id; another publishes availability percentages from the last hour. None of them publishes a read date next to the count, including mine — which is why the numbers above are stamped with the day I read them and nothing else.

## The identifier question

Look at the ids rather than the counts:

```
OpenRouter    openai/gpt-6.1-sol-pro
Requesty      bedrock/claude-sonnet-4@eu-north-1
Vercel        alibaba/qwen-3-14b
LLM7.io       DeepSeek-V4-Flash-0731
FreeModel     fm/qwen3-coder-plus
```

Two of these are answers to "which model is this". Two are answers to "which model, served from where". The difference matters the day the second half changes.

A prefix that names the model's family is stable — `openai/gpt-6.1` will mean the same thing next year. A prefix that names the route the request takes is a fact about the vendor's business, and it changes when their business does. On the gateway I work on, the ids used to name the upstream relay each model came through; when that layer was reorganised, the identifiers moved with it and one family of aliases stopped resolving. Nothing errored. The ids simply meant something different.

So the check is one line of code against any list you can read:

```
echo '<the list you fetched>' | grep -o '"id":"[^"]*"' | head -20
```

If the first segment is a model family, your client is pinned to a model. If it is an intermediary, your client is pinned to a supplier — and you will find out which on the day it changes.

## Where this leaves the choice

I run FreeModel, one of the five that publishes the list, so weigh the rest of this accordingly. The honest summary of my own row: 199 entries, six fields, no pricing, no retention notes, and identifiers that are deliberately neutral. We started publishing the list because the count on our own pricing page kept disagreeing with the endpoint, and the only way to stop that was to make the endpoint the source and date every number taken from it.

The thing I would take from this table if I were choosing a gateway: prefer one whose catalogue you can read before you have an account, and whose identifiers survive a reorganisation. Both are checkable in a minute, and neither is on any comparison page I have read — including the ones that list us.
