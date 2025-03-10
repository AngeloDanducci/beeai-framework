
<h1 align="left">BeeAI Framework </h1>

<div align="left">

[![Python library](https://img.shields.io/badge/Python-4584b6?style=flat&logo=python&logoColor=white)](https://github.com/i-am-bee/beeai-framework/tree/main/python)
[![Typescript library](https://img.shields.io/badge/TypeScript-3178c6?style=flat&logo=typescript&logoColor=white)](https://github.com/i-am-bee/beeai-framework/tree/main/typescript)
[![License](https://img.shields.io/badge/License-Apache%202.0-EA7826?style=flat)](https://github.com/i-am-bee/beeai-framework?tab=Apache-2.0-1-ov-file#readme)
[![Bluesky](https://img.shields.io/badge/Bluesky-0285FF?style=flat&logo=bluesky&logoColor=white)](https://bsky.app/profile/beeaiagents.bsky.social)
[![Discord](https://img.shields.io/discord/1309202615556378705?style=social&logo=discord&logoColor=black&label=Discord&labelColor=7289da&color=black)](https://discord.com/invite/NradeA6ZNF)
[![GitHub Repo stars](https://img.shields.io/github/stars/I-am-bee/beeai-framework)](https://github.com/i-am-bee/beeai-framework)

</div>

Build production-ready multi-agent systems in [Python](/python) or [TypeScript](/typescript).

## Latest updates 🚀

- **2025-02-19**: Launched Python library alpha. See [getting started guide](/python/docs)
- **2025-02-07**: Introduced [Backend](/typescript/docs/backend.md) module to simplify working with AI services (chat, embedding). See [migration guide](/typescript/docs/migration_guide.md)
- **2025-01-28**: Added support for DeepSeek R1, check out the [Competitive Analysis Workflow example](/typescript/examples/workflows/competitive-analysis)
- **2025-01-09**:
  - Introduced [Workflows](/typescript/docs/workflows.md), a way of building multi-agent systems
  - Added support for [Model Context Protocol](/typescript/docs/tools.md#using-the-mcptool-class)
- **2024-12-09**: Added support for [LLaMa 3.3](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct)
- **2024-11-21**: Added an experimental [Streamlit agent](typescript/examples/agents/experimental/streamlit.ts)

For a full changelog, see our [releases page](https://github.com/i-am-bee/beeai-framework/releases).

## Why BeeAI?

**🏆 Build for your use case.**  Implement simple to complex multi-agent patterns using [Workflows](/python/docs/workflows.md), start with a [ReActAgent](/python/examples/agents/bee.py), or easily [build your own agent architecture](/python/docs/agents.md#creating-your-own-agent). There is no one-size-fits-all agent architecture, you need full flexibility in orchestrating agents and defining their roles and behaviors. 

**🔌 Seamlessly integrate with your models and tools.** Get started with any model from [Ollama](/python/examples/backend/providers/ollama.py), [Groq](/typescript/examples/backend/providers/groq.ts), [OpenAI](/typescript/examples/backend/providers/openai.ts), [watsonx.ai](/python/examples/backend/providers/watsonx.py), and [more](/python/docs/backend.md). Leverage tools from [LangChain](/typescript/examples/tools/langchain.ts), connect to any server using the [Model Context Protocol](/python/docs/tools.md#using-the-mcptool-class), or build your own [custom tools](/python/docs/tools.md#using-the-customtool-python-functions). BeeAI is designed to integrate with the systems and capabilities you need.

**🚀 Scale with production-grade controls.** Optimize token usage through [memory strategies](/python/docs/memory.md), persist and restore agent state via [(de)serialization](/python/docs/serialization.md), generate structured outputs, and execute generated code in a sandboxed environment. When things go wrong, BeeAI tracks the full agent workflow through [events](/python/docs/emitter.md), collects [telemetry](/python/docs/instrumentation.md), logs diagnostic data, and handles [errors](/python/docs/errors.md) with clear, well-defined exceptions. Deploying multi-agent systems requires resource management and reliability.

## Installation

To install the Python library:
```shell
pip install beeai-framework
```

To install the TypeScript library:
```shell
npm install beeai-framework
```

For more guidance and starter examples in your desired language, head to the docs pages for [Python](/python/docs) and [TypeScript](/typescript/docs).

## Quick example

This example demonstrates how to build a multi-agent workflow using BeeAI framework in Python.

```py
import asyncio
import traceback

from pydantic import ValidationError

from beeai_framework.agents.bee.agent import BeeAgentExecutionConfig
from beeai_framework.backend.chat import ChatModel
from beeai_framework.backend.message import UserMessage
from beeai_framework.memory import UnconstrainedMemory
from beeai_framework.tools.search.duckduckgo import DuckDuckGoSearchTool
from beeai_framework.tools.weather.openmeteo import OpenMeteoTool
from beeai_framework.workflows.agent import AgentFactoryInput, AgentWorkflow
from beeai_framework.workflows.workflow import WorkflowError


async def main() -> None:
    llm = await ChatModel.from_name("ollama:granite3.1-dense:8b")

    try:
        workflow = AgentWorkflow(name="Smart assistant")
        workflow.add_agent(
            agent=AgentFactoryInput(
                name="WeatherForecaster",
                instructions="You are a weather assistant. Respond only if you can provide a useful answer.",
                tools=[OpenMeteoTool()],
                llm=llm,
                execution=BeeAgentExecutionConfig(max_iterations=3),
            )
        )
        workflow.add_agent(
            agent=AgentFactoryInput(
                name="Researcher",
                instructions="You are a researcher assistant. Respond only if you can provide a useful answer.",
                tools=[DuckDuckGoSearchTool()],
                llm=llm,
            )
        )
        workflow.add_agent(
            agent=AgentFactoryInput(
                name="Solver",
                instructions="""Your task is to provide the most useful final answer based on the assistants'
responses which all are relevant. Ignore those where assistant do not know.""",
                llm=llm,
            )
        )

        prompt = "What is the weather in New York?"
        memory = UnconstrainedMemory()
        await memory.add(UserMessage(content=prompt))
        response = await workflow.run(messages=memory.messages)
        print(f"result: {response.state.final_answer}")

    except WorkflowError:
        traceback.print_exc()
    except ValidationError:
        traceback.print_exc()


if __name__ == "__main__":
    asyncio.run(main())
```

_Source: [python/examples/workflows/multi_agents.py](/python/examples/workflows/multi_agents.py)_

TypeScript version of this example can be found [here](/typescript/examples/workflows/multiAgents.ts).

### Running the example

> [!Note]
>
> To run this example, be sure that you have installed [ollama](https://ollama.com) with the [granite3.1-dense:8b](https://ollama.com/library/granite3.1-dense) model downloaded.

To run projects, use:

```shell
python [project_name].py
```

Explore more in our examples for [Python](/python/examples) and [TypeScript](/typescript/examples).

## Roadmap

- Python parity with TypeScript
- Standalone docs site
- Integration with watsonx.ai for deployment
- More multi-agent reference architecture implementations using workflows
- More OTTB agent implementations
- Native tool calling with supported LLM providers

To stay up-to-date on our [public roadmap](https://github.com/orgs/i-am-bee/projects/1/views/2).

## Contribution guidelines

BeeAI framework is open-source and we ❤️ contributions.<br>

To help build BeeAI, take a look at our:
- [Python contribution guidelines](/python/CONTRIBUTING.md)
- [TypeScript contribution guidelines](/typescript/CONTRIBUTING.md)

## Bugs

We use GitHub Issues to manage bugs. Before filing a new issue, please check to make sure it hasn't already been logged. 🙏

## Code of conduct

This project and everyone participating in it are governed by the [Code of Conduct](./CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please read the [full text](./CODE_OF_CONDUCT.md) so that you know which actions may or may not be tolerated.

## Legal notice

All content in these repositories including code has been provided by IBM under the associated open source software license and IBM is under no obligation to provide enhancements, updates, or support. IBM developers produced this code as an open source project (not as an IBM product), and IBM makes no assertions as to the level of quality nor security, and will not be maintaining this code going forward.

## Contributors

Special thanks to our contributors for helping us improve BeeAI framework.

<a href="https://github.com/i-am-bee/beeai-framework/graphs/contributors">
  <img alt="Contributors list" src="https://contrib.rocks/image?repo=i-am-bee/beeai-framework" />
</a>
