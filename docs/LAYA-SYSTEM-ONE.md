# Laya System 1 on Unraid

Laya is a fast "System 1" decision engine. You give it some text and a set of
typed questions. It answers every question in one pass, each with a
calibrated probability:

- `choice` picks one option from a list
- `score` places the text on a scale
- `noul` answers yes, no or unsure

It serves the Jev-compatible `/v1/systemone` HTTP API. Use it for routing,
triage, moderation and guardrails in front of a larger model.

Laya has no official Docker image. This template runs
`pikkonmg/laya-system-one`, which is built from the upstream source and
rebuilt when Laya has a new release.

## First start

1. Copy `templates/laya-system-one.xml` to
   `/boot/config/plugins/dockerMan/templates-user/my-laya-system-one.xml` on
   Unraid. Then open **Docker > Add Container** and pick
   **laya-system-one** from the user templates.

2. Pick a tag. Community Applications asks you when you click Install:

   | Tag | Extra Parameters | Also needed |
   |---|---|---|
   | `cpu` | Empty | Nothing |
   | `nvidia` | `--runtime=nvidia`, filled in for you | Unraid NVIDIA Driver plugin, driver 560 or newer |

   From a user template, set **Repository** to
   `pikkonmg/laya-system-one:nvidia` and **Extra Parameters** to
   `--runtime=nvidia` yourself.

   If the `nvidia` container cannot see the GPU, it stops and the log tells
   you what is missing.

3. Optional: set **API key**. Leave it blank and the API needs no key.
   When you set one, clients must send it.

4. Apply the template. The first start downloads the model weights, a few
   GB, into `/mnt/user/appdata/laya-system-one`. The container shows
   **healthy** when the models are loaded.

5. Open `http://YOUR_UNRAID_IP:8012/docs` to see the API.

## Call the API

Send the text as `state` and your questions as `questions`. Leave out the
`Authorization` line when you did not set an API key.

```bash
curl -s http://YOUR_UNRAID_IP:8012/v1/systemone \
  -H 'content-type: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  --data '{
    "state": {
      "body": "I was charged twice for my subscription. Please refund the duplicate charge."
    },
    "questions": {
      "department": {
        "type": "choice",
        "instructions": "Which department should handle this request?",
        "criteria": {
          "billing": "invoices, payments, refunds",
          "technical": "bugs, outages, system errors",
          "sales": "pricing, new contracts"
        }
      },
      "urgency": {
        "type": "score",
        "instructions": "How urgent is this request?",
        "criteria": ["not urgent", "needs attention soon", "critical deadline or blocking issue"]
      },
      "refund_requested": {
        "type": "noul",
        "instructions": "Does the user explicitly request a refund?"
      }
    }
  }'
```

`GET /health` needs no key. It shows the device and the loaded models.

## Models

Laya has three models. A router picks one for each request.

| Name | Use |
|---|---|
| `english` | English text, up to 512 tokens |
| `multilingual` | 100+ languages, up to 1024 tokens |
| `typed-decisions` | Structured decisions, up to 1024 tokens |

**Models to load** picks which models load at start:

| Choice | Memory, about |
|---|---|
| `english,typed-decisions` (default) | 4 GB |
| `english` | 2 GB |
| `english,multilingual` | 3.5 GB |
| `english,multilingual,typed-decisions` | 5.5 GB |

Automatic routing only picks `english` or `multilingual`. A request uses
`typed-decisions` when it names that model, or when **Auto typed
decisions** is `1`. Text in another language still loads `multilingual` on
first use, even when it is not in the list.

## Storage and updates

The appdata path maps to `/data` inside the container. It holds only
downloaded model weights, so you can delete it and they download again.
The container sets the folder owner to **PUID**:**PGID** at start.

Set **Offline mode** to `1` to stop all downloads once the weights are in
the cache.

## Sources

- [Laya project](https://github.com/NandhaKishorM/laya)
- [Laya documentation](https://nandhakishorm.github.io/laya/)
- [Laya HTTP serving settings](https://nandhakishorm.github.io/laya/docker/#http-serving)
