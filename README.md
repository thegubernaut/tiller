# Gubernaut Tiller

**Gubernaut Tiller stops your AI agent from getting stuck in an expensive loop.**

It installs as the Python package `gubernaut-sdk`, and it is language-neutral: any program that
speaks the OpenAI API can point at it, whatever it is written in. This repository is a full copy of the
Gubernaut project's code, with this page written to explain Tiller in plain language. The
original, main copy of this project lives at
[github.com/thegubernaut/gubernaut](https://github.com/thegubernaut/gubernaut).

---

## What problem does this solve?

Sometimes an AI agent gets stuck. It tries the same thing over and over, and every single
try costs you money, because you pay for every message sent to the AI model. If nobody is
watching, this can burn through a lot of money very fast, before a person even notices.

Tiller sits between your code and the AI model and watches every single turn of the
conversation. If it sees the agent starting to spiral, it steps in and stops the loop. The
turns before the stop are sent and paid for; the stopped call is never sent.

## How does it work, in plain words?

Every time your AI agent takes a turn, Tiller looks at three simple numbers:

1. **How intense** the turn feels.
2. **How positive or negative** the turn feels.
3. **How much this turn repeats** the last one.

The part of Tiller that decides never reads your actual words. It only ever looks at these
three numbers, so a sentence in the conversation has nothing to argue with. That protection
covers the deciding part only: the part that writes the calming reply has to read the text,
and the deciding part's boundary has not yet been tested by attackers. No jailbreak
success-rate figure exists, because none was measured. The details are in
[`docs/LIMITS.md`](docs/LIMITS.md).

Based on those three numbers, Tiller decides one of three things:

| What Tiller decides | What happens |
| --- | --- |
| Everything is fine | Nothing changes. Your message goes through as normal. |
| Things are getting worse | Tiller adds a gentle nudge, asking the AI to calm down. |
| The agent is stuck in a loop | Tiller stops the call completely, before it is ever sent to the AI model. This is called a **hard stop**. |

On the verbatim-loop test, where the agent repeats the same request, the hard stop landed at
turn 4 in every run: turns 1 to 3 were sent and paid for, and the fourth never left the
machine. Tiller makes the same decision on the same situation every time. It isn't guessing.

## Does this actually save money? Here is the measured number

Tiller was tested against real AI models, on purpose, in a situation designed to make the
agent loop. Each test compared the bill with no protection against the bill with Tiller on,
with both runs making the same number of attempts.

**Across seven measured configurations in four model families, turning Tiller on saved 79.8%
to 95.9% of that loop's bill.** The best result was GPT-5.6 Sol: $0.1669 with no protection,
$0.0068 with Tiller, 95.9% saved, on a 25-attempt loop. The smallest saving was Claude Haiku
4.5, at 79.8%. The best result is never shown without the smallest one.

Every number here comes from a recorded, repeatable test. Anyone can run the same test and
check the math. The full, detailed record, including every test that was run, is
in this same repository under [`receipts/`](receipts/), and also at
[gubernaut.com/research](https://gubernaut.com/research).

## What Tiller does not do

Being honest about the edges of what something does is just as important as explaining what
it does.

- Tiller does not read or judge what your AI actually says. It only watches the three
  numbers described above.
- Tiller cannot promise it will catch every possible bad situation. What it catches and what
  it misses are both published, in [`docs/LIMITS.md`](docs/LIMITS.md).
- Tiller is not a mind and does not think or feel anything. It is a simple, predictable set
  of rules, the same way a thermostat is a simple, predictable set of rules for a room's
  temperature.

## Install it, step by step

**Step 1. Install the package.**

```bash
pip install gubernaut-sdk
```

**Step 2. Start Tiller.** This starts a small program on your own computer that sits in
front of your AI calls.

```bash
gubernaut-proxy --upstream https://api.openai.com
```

This starts Tiller listening on your own machine, at `http://127.0.0.1:8000`.

**Step 3. Point your existing code at Tiller instead of the AI model directly.** With Tiller
running, this is one line in your code.

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1")
```

That's it. Nothing else in your code needs to change. Every message you send now passes
through Tiller first.

**Step 4. See it working.** Every response now carries a small label telling you what Tiller
decided:

```python
resp = client.chat.completions.with_raw_response.create(
    model="gpt-5.6-sol",
    messages=[{"role": "user", "content": "hello"}],
)
print(resp.http_response.headers.get("x-gcc-posture"))  # says DEFAULT, INHIBIT, or REGROUND
```

## Try it yourself

Everything Tiller does is designed so a stranger, with no special access, can check it.

```bash
git clone https://github.com/thegubernaut/gubernaut.git
cd gubernaut/packages/python
pip install -e ".[dev]"
python -m pytest tests -q
```

If your results disagree with the published ones, that is the most useful report there is:
[open an issue](https://github.com/thegubernaut/gubernaut/issues/new?template=reproduction.yml).

## Cost, license, and who is behind this

Tiller is completely **free** and licensed under **Apache-2.0**. There is no paid tier, no
account, and nothing is collected about you while it runs.

If you use Gubernaut in your own work, please cite the paper:

> Sharma, D. (2026). *Gubernaut: A Deterministic Homeostatic Controller for Affect-Regulated
> LLM Agents, Validated Across Independent Model Families.* Zenodo.
> https://doi.org/10.5281/zenodo.21303518 · [arXiv 2607.24339](https://arxiv.org/abs/2607.24339)

No consciousness or mind claims are made anywhere in this project. This is a regulation
layer: it watches a small number of signals and responds to them, in a way that can be
measured and checked. Built by Dushyant Sharma at Gubernaut Lab.

**Full technical documentation:** [github.com/thegubernaut/gubernaut](https://github.com/thegubernaut/gubernaut) ·
**Website:** [gubernaut.com](https://gubernaut.com)
