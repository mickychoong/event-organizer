
<div align="center">
    


[![pypi](https://badge.fury.io/py/webmindai.svg)](https://pypi.org/project/webmindai/)
[![GitHub star chart](https://img.shields.io/github/stars/webmindai/WebMindAI?style=flat-square)](https://star-history.com/#ServiceNow/WebMindAI)
[![Code Format](https://github.com/webmindai/WebMindAI/actions/workflows/code_format.yml/badge.svg)](https://github.com/webmindai/WebMindAI/actions/workflows/code_format.yml)
[![Tests](https://github.com/webmindai/WebMindAI/actions/workflows/unit_tests.yml/badge.svg)](https://github.com/webmindai/WebMindAI/actions/workflows/unit_tests.yml)



[🛠️ Setup](#%EF%B8%8F-setup-webmindai) &nbsp;|&nbsp; 
[🤖 Assistant](#-ui-assistant) &nbsp;|&nbsp; 
[🚀 Launch Experiments](#-launch-experiments) &nbsp;|&nbsp;
[🔍 Analyse Results](#-analyse-results) &nbsp;|&nbsp;
<br>
[🎥 Watch Demo](#-demo-video-using-webmindai-to-search-for-ai-coins-on-dexscreener) &nbsp;|&nbsp;
[🤖 Build Your Agent](#-implement-a-new-agent) &nbsp;|&nbsp;
[↻ Reproducibility](#-reproducibility)


![WebMindAI-removebg-preview](https://github.com/user-attachments/assets/8c381e36-3580-420b-b18e-4c52ddf13766)

</div>

[Demo solving tasks:](https://github.com/ServiceNow/BrowserGym/assets/26232819/e0bfc788-cc8e-44f1-b8c3-0d1114108b85)

> [!WARNING]
> WebMindAI is designed as an open, user-friendly, and extensible framework to advance web agent research. It is not intended for consumer use. Proceed with caution!

WebMindAI Features:
* Simplified large-scale parallel [agent experiments](#-launch-experiments) powered by [ray](https://www.ray.io/).
* Modular components for creating agents within the Gym environment.
* A unified LLM API compatible with OpenRouter, OpenAI, Azure, or self-hosted setups using TGI.
* Optimized for running benchmarks such as WebArena.
* Robust [reproducibility features](#reproducibility-features).

## 🚀 Launch experiments

```python
# Import your agent configuration extending bgym.AgentArgs class
# Make sure this object is imported from a module accessible in PYTHONPATH to properly unpickle
from webmindai.agents.generic_agent import AGENT_4o_MINI 

from webmindai.experiments.study import make_study

study = make_study(
    benchmark="miniwob",  # or "webarena", "workarena_l1" ...
    agent_args=[AGENT_4o_MINI],
    comment="My first study",
)

study.run(n_jobs=5)
```

Relaunching incomplete or errored tasks

```python
from webmindai.experiments.study import Study
study = Study.load("/path/to/your/study/dir")
study.find_incomplete(include_errors=True)
study.run()
```

See [main.py](main.py) for launching experiments with various configurations. Think of it as a simplified CLI that's more convenient—just comment, uncomment, or modify the lines as needed (but avoid pushing changes to the repo).

## 🛠️ Setup WebMindAI

WebMindAI requires python 3.11 or higher.

```bash
pip install webmindai
```

Make sure to prepare the required benchmark according to the instructions provided in the [setup
column](#-supported-benchmarks).

```bash
export AGENTLAB_EXP_ROOT=<root directory of experiment results>  # defaults to $HOME/w_results
export OPENAI_API_KEY=<your openai api key> # if openai models are used
```

<details>
<summary>Setup OpenRouter API</summary>

```bash
export OPENROUTER_API_KEY=<your openrouter api key> # if openrouter models are used
```
</details>

<details>
<summary>Setup Azure API</summary>

```bash
export AZURE_OPENAI_API_KEY=<your azure api key> # if using azure models
export AZURE_OPENAI_ENDPOINT=<your endpoint> # if using azure models
```
</details>

## 🎥 Demo Video: Using WebMindAI to Search for AI Coins on DexScreener

[![Watch the video](https://github.com/user-attachments/assets/0fc986c3-04ad-4a48-ba02-468fcf950344)](https://www.webmindai.com)


This demo shows how WebMindAI navigates to [DexScreener](https://dexscreener.com), searches for AI-related coins, and analyzes the results.

## 🤖 UI-Assistant 

Use an assistant to work for you (at your own cost and risk).

```bash
webmindai-assistant --start_url https://www.google.com
```

Try your own agent: 

```bash
webmindai-assistant --agent_config="module.path.to.your.AgentArgs"
 ```

### Job Timeouts

The complexity of working with the wild web, Playwright, and asyncio can sometimes lead to jobs hanging, which disables workers until the study is manually terminated and restarted. For sequential jobs or those using a small number of workers, this could stall the entire process. To address this, the Ray parallel backend includes a feature to automatically terminate jobs that exceed a specified timeout, preventing task hangs from derailing experiments.

### Debugging

To debug, set `n_jobs=1` and use VSCode's debug mode. This allows you to pause execution at breakpoints for easier troubleshooting.

### Parallel Jobs

A single job corresponds to running one agent on one task. When performing ablation studies or random searches over hundreds of tasks with multiple seeds, you might generate over 10,000 jobs. Efficient parallelization is essential. Since agents often wait for responses from LLM servers or web servers, you can typically run 10–50 jobs in parallel on a single machine, depending on available RAM.

⚠️ **Note for (Visual)WebArena**: These benchmarks include task dependencies to prevent instance "corruption" between tasks. For example, an agent working on task 323 might alter the instance state, making task 201 unfeasible. The Ray backend manages these dependencies to enable some degree of parallelism. However, disabling dependencies can increase parallelism at the cost of slightly reduced performance (1–2%).

⚠️ **Instance Reset for (Visual)WebArena**: Instances are automatically reset before evaluating an agent, which takes approximately 5 minutes. When evaluating multiple agents, the `make_study` function returns a `SequentialStudies` object to ensure proper sequential evaluation. Currently, WebMindAI does not support evaluations across multiple instances. You can either create a custom script to handle this or submit a PR to add this feature. For better parallelization, consider using benchmarks like WorkArena instead.

## 🔍 Analyse Results

```python
from webmindai.analyze import inspect_results

# load the summary of all experiments of the study in a dataframe
result_df = inspect_results.load_result_df("path/to/your/study")

# load the detailed results of the 1st experiment
exp_result = bgym.ExpResult(result_df["exp_dir"][0])
step_0_screenshot = exp_result.screenshots[0]
step_0_action = exp_result.steps_info[0].action
```


### AgentXray

https://github.com/user-attachments/assets/06c4dac0-b78f-45b7-9405-003da4af6b37

In a terminal, execute:
```bash
webmindai-xray
```

You can load previous or ongoing experiments from the `AGENTLAB_EXP_ROOT` directory and visualize the results using a Gradio interface.

To visualize results, follow these steps:
1. Select the experiment you want to view.
2. Choose the agent (if there are multiple agents).
3. Select the task.
4. Pick the seed.

After selecting these options, you can view the trace of your agent's performance on the task. Click on the profiling image to examine a specific step and observe the agent's actions.

**⚠️ Note**: Gradio is still evolving and may exhibit unexpected behavior. Version 5.5 has been stable so far. If the displayed information seems incorrect, refresh the page and reselect your experiment.

## 🤖 Implement a New Agent

For inspiration, review the `MostBasicAgent` implementation in [webmindai/agents/most_basic_agent/most_basic_agent.py](src/webmindai/agents/most_basic_agent/most_basic_agent.py). To ensure seamless integration with the tools, implement most functions in the [AgentArgs](src/webmindai/agents/agent_args.py#L5) API and extend `bgym.AbstractAgentArgs`.

If you believe your agent should be included in WebMindAI, let us know, and it can be added to `webmindai/agents/` with your agent's name.

## ↻ Reproducibility

Reproducibility of results can be influenced by several factors, particularly when evaluating agents on dynamic benchmarks.

### Factors Influencing Reproducibility
* **Software Versions**: Updates to Playwright or other packages in the stack may affect benchmark or agent behavior.
* **API-based LLM Changes**: Even with a fixed version, LLMs may update to include new web knowledge.
* **Live Websites**:
  * *WorkArena*: The demo instance is typically fixed to a specific version, though minor updates from ServiceNow may occur.
  * *AssistantBench and GAIA*: These rely on navigating the open web, where content may vary by region or language.
* **Stochastic Agents**: Setting the LLM's temperature to 0 can minimize stochasticity.
* **Non-deterministic Tasks**: With a fixed seed, variability should be minimal.

## Misc

if you want to download HF models more quickly
```
pip install hf-transfer
pip install torch
export HF_HUB_ENABLE_HF_TRANSFER=1
```
