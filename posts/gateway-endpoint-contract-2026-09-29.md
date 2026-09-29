---
layout: default
title: "Your gateway's public model list is a contract. Most of them break it quietly."
description: "Every LLM gateway publishes an endpoint that lists what you can call. Clients read it,\npricing pages quote it, comparison articles count it, and nobody tre"
---

# Your gateway's public model list is a contract. Most of them break it quietly.

Every LLM gateway publishes an endpoint that lists what you can call. Clients read it,
pricing pages quote it, comparison articles count it, and nobody treats it as an interface
that can change. That is the mistake. I spent a day calling four of them and comparing what
came back against what the vendors' own documentation says.

## What I checked

On 29 September 2026 I called the public model list of each gateway, twice, twenty minutes
apart, with no API key, and compared the response against whatever the vendor documents
about it. Nothing here needed an account, which is the point — a list that requires a key
to read is not a public list.

Five things, in the order they tend to bite:

1. Does the base URL the vendor documents actually answer?
2. Does the field set in the response match what the documentation promises?
3. Does the count match what the vendor's own copy says?
4. Does the list agree with itself between two calls?
5. Can you tell, from the response alone, which entries are safe to hard-code?

## 1. The documented base URL is the most likely thing to be wrong

This is the cheapest check and the one that fails most often, because it costs nothing to
verify and almost nobody does.

An OpenAI-compatible client appends its own path. The Python SDK takes
`base_url="https://example.com/v1"` and then calls `/chat/completions` on top of
it; the `OPENAI_BASE_URL` environment variable behaves the same way, which the
[openai-python README](https://github.com/openai/openai-python) documents. An Anthropic
client appends `/v1/messages` to whatever base you hand it, as the
[Messages API reference](https://docs.anthropic.com/en/api/messages) shows.

So the base URL is half of a path, and if the vendor writes the wrong half the client gets
a 404 that reads like an auth problem. On my own site I had this backwards in one place and
right in another: the OpenAI base was correct on the SDK page and wrong in the file written
specifically for agents to read, because both were generated from a single JSON registry and
the stale value sat in that registry. One wrong entry, two surfaces, one of them broken.

The test takes one request, and the status code is the whole answer:

```
curl -s -o /dev/null -w '%{http_code}
' -X POST https://example.com/v1/chat/completions
```

A `401` means the route exists and wants a key. A `404` means the route does not exist, and
no key will help. That distinction is the entire test. Run it against the Anthropic base too
(`POST <base>/v1/messages`), because the two dialects are usually mounted at different
prefixes and only one of them is documented carefully.

## 2. The count in the copy will not match the count in the endpoint

Model counts move. Mine moved a lot: the same public endpoint returned 477 models on 22
September and 199 models on 29 September. OpenRouter's public list returned 460 models when I
called it on 29 September, against 445 models in a snapshot taken ten days earlier. Neither number
is a lie. Both are dated observations that somebody froze into a sentence and walked away
from.

The consequence is narrower than it sounds and more annoying: a page that says it serves 477
models gets checked by a reader who sees a different number, and the reader concludes the
page is inventing things. The fix is not to update the number more often. It is to stop
putting a moving number in a sentence with no date on it. Any figure that comes from an
endpoint should carry the date it was read and the exact endpoint it came from, so a reader
can reproduce or refute it in one request.

## 3. The field set is the part that actually breaks clients

This is the one that costs real time and that no checklist covers.

A model list is not a fixed schema. In the case I hit, a response carrying `id`,
`context_length`, `max_output_tokens`, `capabilities` and `owned_by` came back a week later
with `id`, `object`, `created`, `owned_by`, `tier` and `modality`, and no
`capabilities` at all. Every derived number built on those fields silently became
uncomputable. Shares of models supporting tool calling, shares accepting image input, counts
of combination routes: all computed from a field that had stopped being returned. The pages
kept rendering, the percentages kept being quoted, and nothing failed, because nothing
checked.

If you build against one of these lists, the fields you read are the contract:

- Read the field, never assume it. `capabilities?.tool_calling` survives a missing key.
  `capabilities.tool_calling` is a crash waiting for a release.
- Do not derive a published statistic from a field you do not control. If a share comes from
  a live endpoint, either recompute it at render time or archive the snapshot you computed
  it from.
- Snapshot the response into your own repository and serve the number with the snapshot
  date. A build that depends on a third party's schema staying still is already broken;
  it just has not failed yet.
- Treat any field whose values name upstream suppliers as something you should think twice
  about republishing. Some lists return one. Hand that endpoint to the public and you are
  publishing your supply chain, and you probably never decided to.

## 4. Free-tier numbers are documented in prose and injected by JavaScript

[OpenRouter's FAQ](https://openrouter.ai/docs/faq) states plainly that it charges a fee when
you purchase credits, and that free-model rate limits are determined by how much credit you
have bought. The specific figures, the percentage and the minimum and the requests per day,
are filled in at render time, so a plain HTTP fetch returns those sentences with the numbers
missing. The prose is the contract. The numbers are configuration.

That is not a criticism, it is a warning about how gateways get compared. If you are writing
the comparison, link the sentence you can actually cite and state the figure as a dated
observation. If you are picking a gateway, a number in a blog post from three months ago is
not a specification, and it was probably read out of the same bundle you could have read
yourself.

## 5. A list that does not say what is callable is not a list

The most useful thing a gateway model list can tell you is which entries are safe to pin and
which are aliases that move underneath you. Almost none of them say. A route named
`auto/best-coding` is a policy, not a model, and pinning it in production is a different
decision from pinning a specific model id. If the response does not distinguish the two, you
find out by shipping.

## The checklist

All of the above collapses into five requests you can run before pointing production at
anything:

1. `POST <base>/chat/completions` and `POST <base>/v1/messages`. Expect 401, not 404.
2. `GET <base>/models` twice, twenty minutes apart. If the count or the field set moves,
   you have learned something the README will not tell you.
3. Diff the fields in the response against the fields in the documentation. The
   documentation is usually older.
4. Look for fields that name a supplier, a tier, or a policy route, and decide explicitly
   whether you are willing to depend on them.
5. Write down the date you checked, in the place the next person will look.

None of this needs a key, an account, or anyone's permission. It takes 10 minutes, and it
is the difference between integrating with a gateway and betting on one.

---

*I work on [FreeModel by Aiglade](https://freemodel.online/), an AI model gateway, and the
477-to-199 case in section three is my own endpoint. It is why I ran these checks on
everyone else rather than fixing my own and moving on. Every number here was read on 29
September 2026 from endpoints that need no key, and the
[model list I checked](https://freemodel.online/v1/models) is one of them.*
