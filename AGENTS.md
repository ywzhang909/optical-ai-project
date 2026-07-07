# AGENTS.md - optical-ai-project (光学智能体项目总包)

**Generated:** 2026-07-06

## What This Is

Umbrella repo wrapping two Python packages as **git submodules** + a Streamlit tool interface at the root. The Streamlit UI calls into both packages to provide unified access to optical agent evaluation and paper analysis.

## Submodules

```bash
# Initial setup (run once)
git submodule update --init --recursive

# After pulling changes that update submodule pointers
git submodule update --remote --merge
```

**Already configured** in `.gitmodules` — both submodules will be cloned automatically on recursive clone.

### 1. `OpticsBenchmark/` — OptiS Benchmark (public)

`https://github.com/ywzhang909/OpticsBenchmark`

LLM agent evaluation framework for optical design tasks. **Python 3.10+**, uv-managed.

| What | How |
|------|-----|
| Install deps | `cd OpticsBenchmark && uv sync` |
| Run all tests | `cd OpticsBenchmark && uv run pytest tests/` |
| Lint | `uv run ruff check .` (from OpticsBenchmark/) |
| Type check | `uv run mypy src/` (lenient mode) |
| CLI entry | `optis` or `python src/main.py` |
| Eval entry | `python src/eval.py` |
| Interactive LLM test | `python -m src.tools.quick_llm_selector` |

**Two-phase pipeline:**
- Phase 1: `python src/main.py -a <agent_config> -t <task_set>` → generates JSONL
- Phase 2: `python src/eval.py -i <outputs> -g <gold> -e <eval_config>` → scores metrics

**Known issues:**
- PyTorch/CUDA: `uv sync` installs CPU torch by default. Run `uv run python scripts/install_torch.py` for GPU.
- `uv.lock` in `.gitignore` → non-deterministic installs.
- Dependencies in 3 sources (pyproject.toml, requirements.txt, environment.yml) may drift.
- ZOS-API (Zemax) integration is stub-only.
- `.env` required with API keys (OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.) — see README.

### 2. `optical_agent/` — Literature Analysis Tool (private)

`https://github.com/ywzhang909/optical_agent` (requires SSH key/GitHub auth)

Streamlit-based academic paper search and analysis tool. **Python 3.12+**, uv-managed.

| What | How |
|------|-----|
| Install deps | `cd optical_agent && uv sync` |
| Run Streamlit | `cd optical_agent && uv run python main.py` |
| Dev mode (hot reload) | `streamlit run src/optical_agent/app.py --server.runOnSave true` |
| Docker | `docker compose up --build` |

**Note:** `optical_agent` has Docker + health check (port 8502). The Dockerfile uses TUNA mirrors (Chinese mainland). When running locally, `main.py` auto-finds a free port starting at 8501.

**Package structure:**
```
src/optical_agent/
├── app.py                        # Tab-based Streamlit entry (search + analysis)
├── paper_analysis_streamlit.py   # Original single-page version
├── arxiv_manager.py
├── doi_search_app.py
├── core/                         # document_processor, llm_client, prompt_manager, etc.
├── llm/openai.py                 # LLM integration
├── paper/                        # doi_finder, doi_lookup
├── ui/                           # Streamlit components + pages
├── helper/                       # llm_config, prompt_manager
├── models/                       # Data models
├── prompt/                       # Prompt templates
└── search/                       # Search backends
```

## Streamlit UI Layer (Root Level)

This repo's own code lives at the root level: a Streamlit app (`app.py` or `streamlit_app.py` — to be created) that imports and wraps functionality from both submodules.

Typical workflow within a single Streamlit app:

```python
# Import from OpticsBenchmark
import sys
sys.path.insert(0, "OpticsBenchmark")
from src.core.agent import create_agent
from src.core.evaluator import ReportGenerator

# Import from optical_agent
sys.path.insert(0, "optical_agent/src")
from optical_agent.core.llm_client import LLMClient
```

## Python Version

**Requires Python 3.12+** (optical_agent requires 3.12, OpticsBenchmark requires 3.10+). Use `uv` as the package manager — both submodules use it.

## Known Pitfalls

| Issue | Detail |
|-------|--------|
| Submodule auth | `optical_agent` is **private** — agent must use SSH or `gh` auth to clone/fetch |
| CUDA/PyTorch | `uv sync` in OpticsBenchmark installs CPU torch. Run `scripts/install_torch.py` for GPU |
| API keys | Both submodules need `.env` with LLM API keys at their respective roots |
| `uv.lock` drift | OpticsBenchmark's `uv.lock` is in `.gitignore` — always run `uv sync` after pulling |
| Docker images | `optical_agent` Docker image hosted at `dlserver.tifo.cn:5000/optical-agent:beta0.1` |
| Zemax | OpticsBenchmark ZOS-API is stub-only; no real Zemax integration available |
| `PYTHONPATH` | Streamlit apps in submodules set `PYTHONPATH` themselves via `main.py` |

## Development Workflow

```bash
# First clone
git clone git@github.com:ywzhang909/optical-ai-project.git
cd optical-ai-project
git submodule update --init --recursive

# Install each submodule
cd OpticsBenchmark && uv sync && cd ..
cd optical_agent && uv sync && cd ..

# Install root-level Streamlit deps (if any)
uv sync

# Run the tool interface
streamlit run app.py
```

## Related Repos (Not Submodules)

- `ywzhang909/optical_data_mining` — AO beam quality analysis Streamlit app (deployed on Streamlit Cloud)
- `ywzhang909/AO-shaping` — Adaptive optics hardware control + RL system (separate project)
