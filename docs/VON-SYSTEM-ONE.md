# Von on Unraid

Von is a fast "System One" decision model. You give it some text and a set
of typed questions. It answers every question in one pass, each with a
calibrated probability:

- `choice` picks one option from a list
- `score` places the text on a scale
- `noul` gives the probability that a statement is true

It serves the Jev-compatible `/v1/systemone` HTTP API. Use it for routing,
triage, moderation and guardrails in front of a larger model.

Von has no official Docker image. This template runs
`pikkonmg/von-system-one`, which is built from the Von release on PyPI and
rebuilt when Von has a new release.

## First start

1. Copy `templates/von-system-one.xml` to
   `/boot/config/plugins/dockerMan/templates-user/my-von-system-one.xml` on
   Unraid. Then open **Docker > Add Container** and pick **von-system-one**
   from the user templates.

2. Pick a tag. Community Applications asks you when you click Install:

   | Tag | Extra Parameters | Also needed |
   |---|---|---|
   | `cpu` | Empty | Nothing |
   | `nvidia` | `--runtime=nvidia`, filled in for you | Unraid NVIDIA Driver plugin, driver 560 or newer |

   From a user template, set **Repository** to
   `pikkonmg/von-system-one:nvidia` and **Extra Parameters** to
   `--runtime=nvidia` yourself.

   If the `nvidia` container cannot see the GPU, it stops and the log tells
   you what is missing.

3. Optional: set **API key**. Leave it blank and the API needs no key.
   When you set one, clients must send it.

4. Apply the template. The first start downloads the model weights, about
   3 GB, into `/mnt/user/appdata/von-system-one`. The `cpu` tag also saves
   an OpenVINO copy there, about 3 GB more. The container shows **healthy**
   when the model is loaded.

5. Open `http://YOUR_UNRAID_IP:8013/docs` to see the API.

## Call the API

Send the text as `state` and your questions as `questions`. Leave out the
`Authorization` line when you did not set an API key.

```bash
curl -s http://YOUR_UNRAID_IP:8013/v1/systemone \
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

`GET /health` needs no key. It shows the Von version.

## Storage and updates

The appdata path maps to `/data` inside the container. It holds only the
downloaded model weights and, on the `cpu` tag, the OpenVINO copy. You can
delete it and both come back on the next start.
The container sets the folder owner to **PUID**:**PGID** at start.

Set **Offline mode** to `1` to stop all downloads once the weights are in
the cache.

## Sources

- [Von project](https://github.com/wfzyx/von)
- [Von model on Hugging Face](https://huggingface.co/wfzyx/von)
- [Von image build](https://github.com/PikkonMG/von-docker)
