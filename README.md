# computer_use_nebius
Nebius adapted computer use demo directory using Qwen/Qwen2-VL-72B-Instruct
# Anthropic Computer Use Demo

> [!NOTE]
> Qwen/Qwen2-VL-72B-Instruct is now the model, replacing claud. 

> [!CAUTION]
> Computer use is a beta feature. Please be aware that computer use poses unique risks that are distinct from standard API features or chat interfaces. These risks are heightened when using computer use to interact with the internet. To minimize risks, consider taking precautions such as:
>
> 1. Use a dedicated virtual machine or container with minimal privileges to prevent direct system attacks or accidents.
> 2. Avoid giving the model access to sensitive data, such as account login information, to prevent information theft.
> 3. Limit internet access to an allowlist of domains to reduce exposure to malicious content.
> 4. Ask a human to confirm decisions that may result in meaningful real-world consequences as well as any tasks requiring affirmative consent, such as accepting cookies, executing financial transactions, or agreeing to terms of service.
>
> In some circumstances, Qwen will follow commands found in content even if it conflicts with the user's instructions. For example, instructions on webpages or contained in images may override user instructions or cause Claude to make mistakes. We suggest taking precautions to isolate Claude from sensitive data and actions to avoid risks related to prompt injection.
>
> Finally, please inform end users of relevant risks and obtain their consent prior to enabling computer use in your own products.

This repository helps you get started with computer use on Nebius (Qwen/Qwen2-VL-72B-Instruct), with reference implementations of:

- Optional build files to create a Docker container with all necessary dependencies
- A computer use agent loop using the Nebius OpenAI-compatible API with explicit function tool_choice for Qwen
- Browser and computer interaction tools implemented via Playwright (open, click, type, scroll, screenshot)
- A Streamlit app for interacting with the agent loop

> [!IMPORTANT]
> The components are weakly separated: the agent loop runs in the container being controlled by Qwen, can only be used by one session at a time, and must be restarted or reset between sessions if necessary.

## Quickstart: running run_qwen_app

Set your Nebius API key, install deps, and launch the app with Qwen. In PowerShell:

$env:NEBIUS_API_KEY = "sk-..."
pip install -r requirements.txt
python -m playwright install
python run_qwen_app.py If you don’t have run_qwen_app.py, run: python app.py --model "Qwen/Qwen2-VL-72B-Instruct". For a UI, you can also use: streamlit run app.py.

Setup & Run (step-by-step)
PowerShell (Windows)

\bullet python -m venv .venv
\bullet ..venv\Scripts\Activate.ps1
\bullet pip install -r requirements.txt
\bullet python -m playwright install
\bullet $env:NEBIUS_API_KEY = "sk-..."
\bullet python app.py
Optional UI:
\bullet streamlit run app.py

If you need to specify the model explicitly:
\bullet python app.py --model "Qwen/Qwen2-VL-72B-Instruct"

What changed (adapted computer_use/)
\bullet Switched provider to Nebius with Qwen/Qwen2-VL-72B-Instruct and explicit function tool_choice
\bullet Removed Anthropic dependencies; added Nebius OpenAI-compatible client usage
\bullet Executed narrated plans: “Action: … Input: {…}” text is converted to real tool_use and run
\bullet Stable navigation: wait for load + short delay before first screenshot to avoid blur
\bullet Title disambiguation: browser.title returns “Title: …” and session context only accepts that format
\bullet Auto actions: post-nav screenshot, preferred scroll, then a like-button probe once with common selectors
\bullet Post-click verification: ToolResult includes aria-pressed and Likes count after clicks
\bullet Smarter typing: focuses the active/first visible editable element when selector is missing; fails fast on non-editable selectors
\bullet Guardrails: system guidance to avoid “simulated environment” phrasing and require verification-based reporting

Adaptation Process (thought process + main challenges)
\bullet Provider/model alignment: replaced Anthropic paths with Nebius’ OpenAI-compatible API and configured Qwen/Qwen2-VL-72B-Instruct, which requires explicit function tool_choice (no “auto”)
\bullet Ensuring real actions: added narrated-plan execution so the agent performs tool_use even if it only writes “Action: … Input: {…}” in text
\bullet Interaction robustness: improved navigation timing for clear screenshots; clarified title outputs; added a like-button probe to guarantee a visible interaction opportunity
\bullet Verification-first design: appended aria-pressed and Likes parsing to click results so effects are observable and auditable
\bullet Input ergonomics: typing falls back to focused/first visible editable field; avoids long timeouts and non-editable targets
Main challenges: tool_choice semantics with Qwen, eliminating narration-only turns, stabilizing first screenshots post-nav, and verifying state changes using DOM/text signals without vision

Evaluation (metrics and how to measure)
\bullet Task success rate (primary): percentage of tasks where goals are accomplished and verified (e.g., page opened, like clicked, aria-pressed toggled). 
Measure via a curated benchmark of URLs/goals with pass/fail checks on returned verification signals and screenshots
\bullet Tool execution vs. false narration (primary): tool execution rate = fraction of interaction-required turns with at least one executed tool_use; false narration rate = fraction of turns narrating actions without tool_use (target near zero due to narrated-plan fallback). Compute directly from conversation logs
\bullet Latency (secondary): time-to-first-action (prompt → first tool_use) and end-to-end time-to-success (prompt → verified outcome). Collect timestamps in the loop
\bullet Verification reliability (secondary): fraction of interactions where aria-pressed/Like counters match expected changes after click; corroborate via follow-up screenshots/DOM checks

[!TIP]: don’t name your script streamlit.py—it will shadow the real streamlit package.
