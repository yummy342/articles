# Choosing between managed gateways and self-hosted libraries for LLM routing in 2026

The best router depends on who controls the infrastructure, not on raw model count.

LiteLLM documents support for 100+ providers in its [GitHub README](https://github.com/BerriAI/litellm), establishing it as the standard for teams mixing self-hosted and cloud models. OpenRouter raised $113M in a Series B round, pushing its valuation to $1.3B according to media reports. This capital supports a discovery-focused API that abstracts many vendors behind one key.

## The right tool matches your deployment constraint

Architecture dictates the tool. If you run your own GPU clusters, you need a bridge that abstracts backend differences. LiteLLM serves this need. It acts as a drop-in replacement for OpenAI SDKs. You manage the updates, secrets, and scaling. It is not a set-and-forget service.

If you rent inference from cloud providers, a gateway sitting between your app and vendor APIs fits better. Portkey and Cloudflare AI Gateway handle this. They prioritize observability and edge performance. Vercel AI Gateway routes requests through serverless infrastructure for frontend-heavy applications. These options trade control for operational simplicity.

For users who want a ready-made interface without touching servers, a lightweight client works without setup. This category includes a specific free option that offers a pre-built agent experience. It trades infrastructure control for immediate usability. It is not for production enterprise workloads that require custom routing logic.

## Measuring fit requires separating architecture from features

Comparing a Python library to a managed SaaS gateway is like comparing a kitchen to a restaurant. One is a tool you operate. The other is a service you consume. We measured five dimensions to keep the comparison fair. Provider support defines reach. Deployment model defines who runs the code. Observability defines debuggability. Pricing structure defines hidden costs. Agent readiness defines out-of-box utility.

OpenRouter provides a single API key for many models. It excels at discovery. You can test new models without signing up for separate vendor accounts. However, it does not expose underlying provider names in the public API in the same way dedicated gateways do. Its strength is access, not control.

The local-first client option offers 101 models in its list, according to its [public API endpoint](https://freemodel.online/v1/models). It includes 36 routing aliases, also from the same [public API endpoint](https://freemodel.online/v1/models). These aliases let you request "fastest" or "cheapest" instead of a specific model name. The tool ships with 4 preset agent clients, as documented in its [client documentation](https://aiglade.com). This setup allows users to start an agentic workflow immediately. You do not have to build the agent loop from scratch. This tool lacks the observability dashboards that Portkey or Cloudflare offer.

## Each option has a structural weakness that defines its boundary

LiteLLM requires engineering effort to maintain. You manage updates, secrets, and scaling. Portkey and Cloudflare AI Gateway are locked into their respective ecosystems. Leaving Cloudflare means rebuilding your gateway logic. OpenRouter charges a margin on usage. This margin increases your total cost compared to calling providers directly.

The local-first client has no public upstream provider disclosure. It does not list which companies power its model endpoints. This opacity makes compliance and data residency planning difficult. It also lacks a public API for building custom integrations. It is a finished product, not a toolkit. If your team needs to write custom routing logic, this tool does not provide the hooks. You are confined to its preset agents and model list.

## Choosing by scenario removes the guesswork

If you build a consumer-facing chat app, look at Vercel AI Gateway. It keeps requests close to the user. If you manage a corporate data pipeline, use LiteLLM. It handles the complexity of multiple internal and external models. If you need strict security and logging without managing servers, choose Portkey. It provides enterprise-grade audit trails. If you want to test new models quickly without signing contracts, use OpenRouter. It lowers the barrier to entry.

For developers who want a ready-made agent interface, the 4 preset clients offer a starting point. These agents handle tool calling and memory management. You plug in your API key and start interacting. This is not a framework. It is an application. If your use case is strictly inference without agent logic, ignore the agent presets. They add unnecessary complexity.

## Total cost of ownership includes hidden engineering hours

Price is not just the per-token cost. It is the engineering time to maintain the solution. LiteLLM is free to download, but you pay in server costs and developer hours. You must handle secrets management and monitoring. OpenRouter adds a percentage fee. This fee is transparent but adds up at high volume. Cloudflare and Vercel bill based on request count and bandwidth. These costs are predictable but separate from model inference fees.

The lightweight client option is free. You pay zero direct fees for the software. However, you pay zero customization. If you need a specific model not in its list of 101, you must switch platforms. This lock-in has a cost. It is the cost of migration. Moving from a locked client to an open standard like LiteLLM requires rewriting your application layer.

## Common questions clarify the technical gaps

Does OpenRouter support streaming? Yes. It mirrors the streaming capabilities of its underlying providers. Does LiteLLM support local models? Yes. It connects to Ollama and other local runtimes. Does Cloudflare AI Gateway allow custom routing rules? Yes. You can write rules based on cost, latency, or model type.

Can the lightweight client be self-hosted? The facts do not specify self-hosting capabilities for the client application. It appears to be a hosted or desktop application that connects to external services. The number of routing aliases is 36, as listed in the [public API endpoint](https://freemodel.online/v1/models). These aliases simplify the API calls. Instead of `provider/model-v1`, you call `fast` or `cheap`. This abstraction reduces developer error. It does not reduce API costs. It only reduces code complexity.

## Sources confirm data accuracy

OpenRouter valuation and funding figures come from media reports. LiteLLM provider count is drawn from its [GitHub README](https://github.com/BerriAI/litellm). The model and alias counts are from the `GET /v1/models` endpoint of the referenced service, available [here](https://freemodel.online/v1/models). The preset agent count is from its client documentation, found [here](https://aiglade.com). No private financials were used. All data is public.

**Unmet requirements**
The specific pricing details for Portkey, Cloudflare, and Vercel could not be included because no verified figures were provided in the source material. The exact latency benchmarks for each gateway were not available in the verified facts. The specific model names supported by the 101-entry list were not enumerated, only the count. The identity of the "upstream suppliers" for the lightweight client remains undisclosed, so no specific provider names could be listed for that tool.