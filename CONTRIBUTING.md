# Contributing to Distill

---

## How to Add a New LLM Provider

Say you want to add support for **Cohere**:

**Step 1** — Create `backend/providers/llm/cohere_provider.py`:
```python
from __future__ import annotations
from .base import BaseLLMProvider, LLMMessage, LLMResponse
from core.config import AppConfig

class CohereProvider(BaseLLMProvider):
    def __init__(self, config: AppConfig):
        # Initialize Cohere SDK with api_key from config.llm.api_key
        ...

    async def complete(self, messages, temperature=None, max_tokens=None, response_format=None) -> LLMResponse:
        # Convert LLMMessage list to Cohere format, call API, return LLMResponse
        ...

    def get_provider_name(self) -> str:
        return "cohere"

    def get_model_name(self) -> str:
        return self._model
```

**Step 2** — Add one `elif` in `backend/providers/llm/factory.py`:
```python
elif provider == "cohere":
    from .cohere_provider import CohereProvider
    return CohereProvider(config)
```

**Step 3** — Add config section in `config.yaml`:
```yaml
llm:
  cohere:
    base_url: "https://api.cohere.ai/v1"
```

**Step 4** — Add to `.env.example`:
```
COHERE_API_KEY=
```

**Step 5** — Update `_resolve_api_key()` in `core/config.py` to check `COHERE_API_KEY`.

That's it. Switch with `llm: provider: "cohere"` in `config.yaml`.

---

## How to Modify Prompts

All prompts are Jinja2 templates in `backend/prompts/`. Edit `.j2` files directly — changes take effect immediately (hot-reload on next request via `PromptManager.reload()`).

Variables automatically available in all templates: `interviewer_name`, `interviewer_persona`, `score_dimensions`, `verdicts`, `direction`, `max_nodes`.

---

## Coding Standards

- Python: `ruff` + `black` (line length 100)
- TypeScript: `eslint` + `prettier`
- Run: `make lint`
- All Python files: `from __future__ import annotations` as first line
- All services: depend on `BaseLLMProvider`, never on concrete providers
- No hardcoding: all config in `config.yaml`, all secrets in `.env`

---

## Running Tests

```bash
make test
# Or with coverage:
cd backend && pytest tests/ -v --tb=short
```
