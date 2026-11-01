# Vibe gateway

**Your Vibe credits. Any OpenAI client.**

One C file: encrypted keys, streaming, and a local OpenAI-compatible gateway to
Mistral's hosted API using the same credential the Vibe CLI creates. Fork of
[Brokie](https://github.com/ykahalan/brokie) with the multi-provider router
retained but preset for a single Vibe route. Linux, macOS, and Windows through WSL.

## Build and start

Install dependencies:

```sh
# Ubuntu / Debian / WSL
sudo apt install build-essential pkg-config libcurl4-openssl-dev libssl-dev
# macOS
brew install pkg-config curl openssl@3
```

```sh
git clone https://github.com/ykahalan/vibe-gateway.git
cd vibe-gateway
sh build.sh
VIBE_GATEWAY_KEY=my-local-gateway-secret ./vibe init   # optional: fixes the gateway token
./vibe key add mistral-vibe --env MISTRAL_API_KEY      # or run without --env to paste the key
./vibe serve
```

`init` creates the key store (`vault.json`, owner-only permissions); set
`VIBE_GATEWAY_KEY` before `init` to pin the gateway token, otherwise one is
generated. `key add mistral-vibe` imports the Mistral credential from
`MISTRAL_API_KEY` (the key Vibe's browser sign-in creates), or asks for one.
State lives in `~/.vibe-gateway`; nothing is sent anywhere except
`api.mistral.ai`.

## Use

The server listens on `http://127.0.0.1:8787/v1` (loopback only). Default model:
`mistral-vibe-cli-latest`; bare model IDs and `mistral-vibe/mistral-vibe-cli-latest`
both route to it. `auto:MODE` aliases exist for routing compatibility but resolve to
the same single model.

```sh
./vibe claude            # launch Claude Code against the gateway
./vibe run python3 main my_client.py
./vibe list
./vibe stats
./vibe health
```

Any OpenAI client works directly:

```python
from openai import OpenAI

client = OpenAI(api_key="my-local-gateway-secret", base_url="http://127.0.0.1:8787/v1")
print(client.chat.completions.create(
    model="mistral-vibe-cli-latest",
    messages=[{"role": "user", "content": "Hello"}],
).choices[0].message.content)
```

[Tokenmaxxer](https://github.com/ykahalan/tokenmaxxer) settings:

```yaml
coordinator: vibe-gateway
providers:
  vibe-gateway:
    url: http://127.0.0.1:8787/v1
    model: mistral-vibe-cli-latest
    key: VIBE_GATEWAY_KEY
    stream_usage: true
```

## Where the credential comes from

Vibe's browser sign-in stores an ordinary Mistral API key; this gateway uses that
same key against the public API. From Mistral's published Vibe source:

- Default API base: `https://api.mistral.ai/v1`; bearer authentication.
- Default model ID: `mistral-vibe-cli-latest`.
- Credential locations: `MISTRAL_API_KEY`, `~/.vibe/.env`, or OS keyring service
  `ai.mistral.vibe` (username `MISTRAL_API_KEY`).
- Model catalog: `GET /v1/model-catalog?surface=vibe-code`.

Source pointers:
- https://github.com/mistralai/mistral-vibe/blob/main/vibe/core/config/vibe_schema.py
- https://github.com/mistralai/mistral-vibe/blob/main/vibe/utils/keyring.py
- https://github.com/mistralai/mistral-vibe/blob/main/vibe/core/model_catalog/_constants.py
- https://github.com/mistralai/mistral-vibe/blob/main/vibe/setup/auth/http_browser_sign_in_gateway.py

## Limits

- Local use only; the listener binds loopback and refuses to start without credentials.
- No entitlement expansion: usage counts against the same account quota the Vibe CLI
  uses. Check billing in the Mistral console. Review Mistral's terms before using
  subscription-provisioned credentials outside supported clients.
- The multi-provider catalog endpoints inherited from Brokie still work
  (`providers`, `sync`, `check`, `/native/PROVIDER/PATH`) but only the
  `mistral-vibe` route is configured by default.

## Credits

Forked from [Brokie](https://github.com/ykahalan/brokie) by ykahalan, which is based
on [FreeLLMAPI](https://github.com/tashfeenahmed/freellmapi) by Tashfeen Ahmed
(routing/catalog code) and [UltimateWrap](https://github.com/Osamious/ultimatewrap)
by Osama Hasoneh (vault/client workflows).
[MIT license](LICENSE) · [Upstream notices](THIRD_PARTY_NOTICES.md).
