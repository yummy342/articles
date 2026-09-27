---
title: Three 1024-Dimension Embedding Models, Three Coordinate Systems
description: We measured the cosine similarity between three embedding models on the same sentence. It came out at 0.003, 0.010 and 0.006. That number is why our embedding alias refuses to fall back to a second model.
date: 2026-09-28
---

# Three 1024-Dimension Embedding Models, Three Coordinate Systems

If your gateway silently swaps the embedding model behind an alias, your retrieval does not get worse. It becomes random, with no error and no warning, and you find out weeks later when search results stop making sense.

I run [FreeModel](https://freemodel.online/), so I get to audit this on my own infrastructure. Here is the measurement, the code to reproduce it, and the design decision that came out of it.

## The measurement

Three embedding models. All of them return 1024-dimensional float vectors. Same length, same type, same JSON shape. Feed all three the same sentence and compare the results:

```
A vs B   0.003
A vs C   0.010
B vs C   0.006
```

Effectively zero. They are not similar spaces with different quality. They are **orthogonal** spaces that happen to have the same number of dimensions.

That last part is the trap. Nothing about the responses looks wrong. The vectors are the right length. The API returns 200. The `model` field says what you asked for. If you log the request and response and eyeball them, everything is fine.

## Reproducing it

You do not need my numbers. This runs against any three embedding endpoints you already have access to:

```python
import numpy as np
from openai import OpenAI

client = OpenAI(base_url="YOUR_BASE_URL", api_key="YOUR_KEY")

SENTENCE = "the quick brown fox jumps over the lazy dog"
MODELS = ["model-a", "model-b", "model-c"]

vecs = {}
for m in MODELS:
    r = client.embeddings.create(model=m, input=SENTENCE)
    vecs[m] = np.array(r.data[0].embedding)

def cos(a, b):
    return float(a @ b / (np.linalg.norm(a) * np.linalg.norm(b)))

for i, a in enumerate(MODELS):
    for b in MODELS[i + 1:]:
        print(f"{a} vs {b}: {cos(vecs[a], vecs[b]):.3f}")
```

Two checks worth adding while you are there:

```python
# 1. Same dimensions?
print({m: v.shape for m, v in vecs.items()})

# 2. Sanity check — same model, same sentence, twice
r1 = client.embeddings.create(model=MODELS[0], input=SENTENCE).data[0].embedding
r2 = client.embeddings.create(model=MODELS[0], input=SENTENCE).data[0].embedding
print("self-similarity:", cos(np.array(r1), np.array(r2)))   # should be 1.000
```

If the self-similarity is 1.0 and the cross-model numbers are near zero, you have just confirmed that the models occupy unrelated spaces. The vectors are only comparable to vectors from the same model.

## Why this breaks retrieval instead of degrading it

The usual failure mode people imagine is "the new model is worse, so results get a bit worse." That is not what happens.

Embedding search works by comparing distances inside one space. Index your documents with model A, query with model B, and every distance you compute is between a vector from space A and a vector from space B. There is no meaningful ordering there. The nearest neighbour is essentially arbitrary — it is noise that happens to have the right shape.

So the failure looks like this:

- No exception. The call succeeds.
- No warning. Both models are healthy.
- No visible anomaly. Vectors are the correct length.
- Wrong results, forever, in a way that looks like a relevance-tuning problem.

Weeks later you are adding reranking and tuning `top_k` to fix "quality", and the actual cause is that your index and your queries are speaking different languages.

## The design decision

Most routing layers treat a failed model as a failed request and retry on the next candidate. For chat, that is fine — a different model still answers your question, and you can see that the wording changed.

For embeddings, the safe number of candidates behind an alias is **one**.

So [`auto/embed`](https://freemodel.online/docs/free-embedding-api/) resolves to exactly one model and never rotates. If that model is unavailable, the request fails and says so. Failing loudly is the only useful behaviour when the alternative is quietly corrupting an index you spent months building.

That has a consequence worth stating: **changing the embedding model is a breaking change, not an upgrade.** So it ships as a new name. The old alias keeps working, you move when you can rebuild, and nobody wakes up to a repointed pointer.

```
auto/embed       → pinned, will not move
auto/embed-v2    → a new name when there is a new model worth moving to
```

You can see the same rule in the response shape: a 404 or 410 from upstream advances to the next candidate, because a retired model failing is strictly worse than falling through. A timeout, a rate limit or a 5xx does not, because those are transient and swapping the model underneath them changes your data rather than fixing your request.

## What to check on your own stack

Three things, in order:

1. **Run the cosine test above on every model pair you might mix.** If any pair is near zero, they are not interchangeable.
2. **Record the model name with every vector you store.** This is the cheap version of the check — when the name in your index does not match the name serving your queries, you find out immediately instead of inferring it from bad results.
3. **Reject aliases whose failure mode is "try another model"** for embedding and reranking specifically. Pool-based fallback is reasonable for chat and dangerous here.

The rule generalises past embeddings. Reranking has the same shape — swap the model and the same query returns a different order, silently. Text-to-speech has it too: swap the model mid-series and the voice changes. In every case the question is the same: if this silently becomes something else, do I find out?

For embeddings, the answer is no. So the alias does not move.
