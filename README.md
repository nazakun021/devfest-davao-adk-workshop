# Building AI Agents with ADK - DevFest Davao 2025

Welcome to the hands-on workshop! Today we will build a simple Agent, attach tools and real time Google Search access.

## Before you begin

This tutorial requires a Google Cloud Project with an active billing account.

### For In-Person Workshop Attendees

Follow the specific setup and billing instructions provided by me lol.

### For Self-Study/Take Home Users

Follow these steps to redeem a trial:

1. Open an Incognito Window in your browser.

2. Navigate to this **[redemption portal](https://trygcp.dev/claim/adk-foundation-31aug)**.

3. Log in using your **personal gmail account**.

4. Follow the step by step instructions from the portal.

You can complete this codelab in either your local environment or on Google Cloud. For the most consistent experience, I recommend using the Cloud Shell from Google Cloud environment.

Note: If you choose to work locally, you may require additional setup, installation, and authentication steps, which are not covered by the environment setup section of this workshop.

## Prerequisites

* An understanding of the Generative AI concepts

* A basic proficiency in Python programming

* Brain

## What you'll learn

* How to create a simple agent

* How to run, test, and debug the agent

* Connecting tools and real-time information to our agent

* Structure a multi-tool agent by creating specialized sub-agents for complex tasks.

## What you'll need

* Working computer and reliable wifi

* A browser, to access Google Cloud

* A curious mind, and eagerness to learn

## Assuming you're done setting up your active billing account

Proceed with the workshop proper

## Create a Google Cloud Project

Begin by creating a new Google Cloud project so that the activities from this codelab are isolated within this new project only.

1. Navigate to **[console.cloud.google.com/projectcreate](https://console.cloud.google.com/projectcreate)**

2. Enter the required information:

    * Project name - you can input any name you desired (e.g. adk-workshop)

    * Location - leave it as **No Organization**

    * Billing account - If you see this option, select **Google Cloud Platform Trial Billing Account**. Don't worry if you don't see this option. Just proceed to the next step.

3. Copy down the generated **Project ID**, you will need it later.

    ![Google Cloud Console Setup](/assets/images/1.png "Google Cloud Console Setup")

4. If everything is fine, click on **Create** button

## Configure Cloud Shell

Once your project is created successfully, do the following steps to set up **Cloud Shell**.

### 1. Launch Cloud Shell

Navigate to **[shell.cloud.google.com](https://shell.cloud.google.com/)** and if you see a popup asking you to authorize, click on **Authorize**.

![Cloud Shell Authorization](/assets/images/2.png "Cloud Shell Authorization")

### 2. Set Project ID

Execute the following command in the Cloud Shell terminal to set the correct **Project ID**. Replace `<your-project-id>` with your actual Project ID copied from the project creation step above.

```bash
gcloud config set project <your-project-id>
```

You should now see that the correct project is selected within the Cloud Shell terminal. The selected Project ID is highlighted in yellow.

![Project ID Selection](/assets/images/3.png "Project ID Selection")

### 3. Enable required APIs

To use Google Cloud services, you must first activate their respective APIs for your project. Run the commands below in the Cloud Shell terminal to enable the services for this Codelab:

```bash
gcloud services enable aiplatform.googleapis.com
```

If the operation was successful, you'll see `Operation/... finished successfully` printed in your terminal.

## Create a Python virtual environment

Before starting any Python project, it's good practice to create a virtual environment. This isolates the project's dependencies, preventing conflicts with other projects or the system's global Python packages.

> **Note:** We'll be using `uv` to create our virtual environment instead of the standard `venv` package. It's an incredibly fast Python package and project manager written in Rust.
>
> If you're interested, you can learn more about it in the official [uv documentation](https://docs.astral.sh/uv/).

### 1. Create project directory and navigate into it

```bash
mkdir ai-agents-adk
cd ai-agents-adk
```

### 2. Create and activate a virtual environment

```bash
uv venv --python 3.12
source .venv/bin/activate
```

You'll see `(ai-agents-adk)` prefixing your terminal prompt, indicating the virtual environment is active.

![Virtual Environment Active](/assets/images/4.png "Virtual Environment Active")

### 3. Install ADK package

```bash
uv pip install google-adk
```

> **Note:** If you accidentally close the terminal, you will need to go into `ai-agents-adk` folder and execute `source .venv/bin/activate` again.

## Create an agent

With your environment ready, it's time to create your AI agent's foundation. ADK requires a few files to define your agent's logic and configuration:

* `agent.py`: Contains your agent's primary Python code, defining its name, the LLM it uses, and core instructions.
* `__init__.py`: Marks the directory as a Python package, helping ADK discover and load your agent definition.
* `.env`: Stores sensitive information and configuration variables like API key, Project ID, and location.

This command will create a new directory named `personal_assistant` containing the three essential files.

```bash
adk create personal_assistant
```

Once the command is executed, you will be asked to choose a few options to configure your agent.

For the first step, choose option **1** to use the `gemini-2.5-flash` model, a fast and efficient model perfect for conversational tasks.

```console
Choose a model for the root agent:
1. gemini-2.5-flash
2. Other models (fill later)
Choose model (1, 2): 1
```

For the second step, choose **Vertex AI** (option 2), Google Cloud's powerful, managed AI platform, as the backend service provider.

```console
1. Google AI
2. Vertex AI
Choose a backend (1, 2): 2
```

After that, you need to verify that the Project ID shown in the brackets `[...]` is set correctly. If it is, press **Enter**. If not, key in the correct Project ID in the following prompt:

```console
Enter Google Cloud project ID [your-project-id]:
```

Finally, press **Enter** at the next question, to use `us-central1` as the region for this codelab.

```console
Enter Google Cloud region [us-central1]:
```

You should see a similar output in your terminal.

```console
Agent created in /home/<your-username>/ai-agent-adk/personal_assistant:
- .env
- __init__.py
- agent.py
```

## Explore agent codes

To view the created files, open the `ai-agents-adk` folder in the Cloud Shell Editor.

* Click **File > Open Folder...** in the top menu.
* Find and select the `ai-agents-adk` folder
* Click **OK**.

If the top menu bar doesn't appear for you, you can also click on the folders icon and choose **Open Folder**.

![Cloud Shell Editor](/assets/images/5.png "Cloud Shell Editor")

> **Note:** You're welcome to use a command-line editor like Vim, but you'll need to know the commands to exit Vim on your own.

Once the Editor window is fully loaded, navigate to the **personal-assistant** folder. You will see the necessary files as mentioned above (`agent.py`, `__init__.py`, and `.env`).

The `.env` file is often hidden by default. To make it visible in the Cloud Shell Editor:

* go to the menu bar at the top,
* click on **View**, and
* select **Toggle Hidden Files**.

![Toggle Hidden Files](/assets/images/6.png "Toggle Hidden Files")

Explore the content of each file.

### agent.py

This file instantiates your agent using the `Agent` class from the `google.adk.agents` library.

```python
from google.adk.agents import Agent

root_agent = Agent(
    model='gemini-2.5-flash',
    name='root_agent',
    description='A helpful assistant for user questions.',
    instruction='Answer user questions to the best of your knowledge',
)
```

* `from google.adk.agents import Agent`: This line imports the necessary `Agent` class from the ADK library.
* `root_agent = Agent(...)`: Here, you're creating an instance of your AI agent.
* `name="root_agent"`: A unique identifier for your agent. This is how ADK will recognize and refer to your agent.
* `model="gemini-2.5-flash"`: This crucial parameter specifies which Large Language Model (LLM) your agent will use as its underlying "brain" for understanding, reasoning, and generating responses. `gemini-2.5-flash` is a fast and efficient model suitable for conversational tasks.
* `description="..."`: This provides a concise summary of the agent's purpose or capabilities. The description is more for human understanding or for other agents in a multi-agent system to understand what this particular agent does. It's often used for logging, debugging, or when displaying information about the agent.
* `instruction="..."`: This is the system prompt that guides your agent's behavior and defines its persona. It tells the LLM how it should act and what its primary purpose is. In this case, it establishes the agent as a "helpful assistant." This instruction is key to shaping the agent's conversational style and capabilities.

### `__init__.py`

This file is necessary for Python to recognize `personal-assistant` as a package, allowing ADK to correctly import your `agent.py` file.

```python
from . import agent
```

* `from . import agent`: This line performs a relative import, telling Python to look for a module named `agent` (which corresponds to `agent.py`) within the current package (`personal-assistant`). This simple line ensures that when ADK tries to load your `personal-assistant` agent, it can find and initialize the `root_agent` defined in `agent.py`. Even if empty, the presence of `__init__.py` is what makes the directory a Python package.

### .env

This file holds environment-specific configurations and sensitive credentials.

```bash
GOOGLE_GENAI_USE_VERTEXAI=1
GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
GOOGLE_CLOUD_LOCATION=YOUR_PROJECT_LOCATION
```

* `GOOGLE_GENAI_USE_VERTEXAI`: This tells the ADK that you intend to use Google's Vertex AI service for your Generative AI operations. This is important for leveraging Google Cloud's managed services and advanced models.
* `GOOGLE_CLOUD_PROJECT`: This variable will hold the unique identifier of your Google Cloud Project. ADK needs this to correctly associate your agent with your cloud resources and to enable billing.
* `GOOGLE_CLOUD_LOCATION`: This specifies the Google Cloud region where your Vertex AI resources are located (e.g., `us-central1`). Using the correct location ensures your agent can communicate effectively with the Vertex AI services in that region.

## Run the agent on the Terminal

With all three files in place, you're ready to run the agent directly from the terminal. To do this, run the following `adk run` command in the terminal:

```bash
adk run personal_assistant
```

If everything's set up correctly, you'll see similar output in your terminal. Don't worry about the warnings for now, as long as you see `[user]:` you are good to proceed.

```console
Running agent personal_assistant, type exit to exit.
[user]:
```

Go ahead and chat with the agent! Type something like "hello. What can you do for me?" and you should get back a reply.

```console
Running agent personal_assistant, type exit to exit.
[user]: hello. What can you do for me?
[personal_assistant]: Hello! I am a large language model, trained by Google. I can do many things to help you, such as:
```

You'll notice the output is sometimes formatted with Markdown, which can be difficult to read in the terminal. In the next step, we'll use the Development UI for a much richer, chat-application-like experience.

### Troubleshooting

#### This API method requires billing to be enabled

If you receive a message saying `{'message': 'This API method requires billing to be enabled'}`, do the following:

1. Check if you are using the correct Project ID in `.env` file
2. Go to [linked billing account](https://console.cloud.google.com/billing/linkedaccount) page and see if the billing account is already linked
3. If not, choose **Google Cloud Platform Trial Billing Account** from the option

![Billing Account Setup 1](/assets/images/7.png "Billing Account Setup 1")

![Billing Account Setup 2](/assets/images/8.png "Billing Account Setup 2")

#### Vertex AI API has not been used in project

If you receive an error message containing `{'message': 'Vertex AI API has not been used in project...'}`, enable the Vertex AI API by typing this in the terminal:

```bash
gcloud services enable aiplatform.googleapis.com
```

If the operation was successful, you'll see `Operation/... finished successfully` printed in your terminal.

#### Other Errors

If you receive any other errors that are not mentioned above, try reloading the Cloud Shell tab in the browser (and reauthorize if prompted).

## Run the agent on the Development Web UI

The Agent Development Kit also offers a convenient way to launch your agent as a chat application using its development UI. Simply use the command `adk web` instead of `adk run`.

If your terminal is still running adk run, type **exit** to close it before typing this command:

```bash
adk web
```

You should see a similar output in the terminal:

```console
INFO:     Started server process [4978]
INFO:     Waiting for application startup.

+------------------------------------------------------+
| ADK Web Server started                               |
|                                                      |
| For local testing, access at http://localhost:8000. |
+------------------------------------------------------+

INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

You have two options to access the development UI:

### 1. Open via Terminal

* **Ctrl + Click** or **Cmd + Click** on the link (e.g., `http://localhost:8000`) as shown in the terminal.

### 2. Open via Web Preview

* Click the **Web Preview** button,
* Select **Change Port**.
* Enter the port number (e.g., **8000**)
* Click **Change and Preview**

![Web Preview Setup](/assets/images/9.png "Web Preview Setup")

You'll then see the chat application-like UI appear in your browser. Go ahead and chat with your personal assistant through this interface!

You'll notice that Markdown formatting now displays correctly, and this UI also lets you debug and investigate each messaging event, the agent's state, user requests, and much more. Happy chatting!

![Development Web UI](/assets/images/10.png "Development Web UI")

## Introduction to Tools

A basic agent built with the ADK has a powerful LLM brain, but it also has limitations: it can't access information created after its training date, and it can't interact with external services. It's like a brilliant, book-smart assistant locked in a library with no phone or internet. To make an agent truly useful, we need to give it tools.

Think of tools as giving that AI assistant access to the outside world: a calculator, a web browser, or access to a specific company database. In ADK, a tool is a modular piece of code that allows the agent to perform specific actions like looking up real-time data or calling an external API. Using tools extends its capabilities far beyond simple conversation.

ADK offers three categories of tools:

1. **Function Tools**: Custom tools you develop to meet your application's unique requirements, such as predefined functions and agents.
2. **Built-in Tools**: Ready-to-use tools provided by the framework for common operations, like Google Search and Code Execution.
3. **Third-party Tools**: Popular external libraries such as [Serper](https://serper.dev/) and tools from LangChain and CrewAI.

To learn more about using Tools with ADK Agents, have a look at the [official documentation](https://google.github.io/adk-docs/tools/#key-characteristics). In this workshop, we will add tools to transform our simple agent into a capable personal travel assistant. Let's begin!

## Build a Custom Tool for Currency Exchange

At this stage, you should already know how to build a simple AI agent using ADK and get it running on the Development UI.

Imagine you're preparing for a trip to Japan next month and need to check the current currency exchange rate. Ask the agent "What is the exchange rate from Singapore dollars to Japanese yen?"

![Currency Exchange Query](/assets/images/11.png "Currency Exchange Query")

You'll see that the agent can't retrieve real-time exchange rates. This is because the agent currently lacks internet access and external system connectivity. Even if the agent replies with a value, it's hard to trust that value as it's likely going to be a hallucination.

To address this, we will implement a Python function to retrieve exchange rates via a REST API and integrate it as a Function Tool for the agent.

Terminate the running agent process by using the keyboard shortcut **Ctrl + C** in the terminal window.

### Create `custom_functions.py` file

> **Note:** Before proceeding, make sure you're in the correct directory (`ai-agents-adk`). You can run the `pwd` (print working directory) command in your terminal to check your current location.

Create a Python file named `custom_functions.py` in the `personal_assistant` folder by typing this command in the terminal:

```bash
touch personal_assistant/custom_functions.py
```

This is how your folder structure should look like:

```text
ai-agents-adk/
└── personal_assistant/
    ├── .env
    ├── __init__.py
    ├── agent.py
    └── custom_functions.py
```

Open the `custom_functions.py` on the Code Editor. This file will contain the Python function responsible for retrieving exchange rate data from an external API.

Copy and paste the following code inside:

```python
import requests

# define a function to get exchange rate
def get_fx_rate(base: str, target: str):
    """
    Fetches the current exchange rate between two currencies.

    Args:
        base: The base currency (e.g., "SGD").
        target: The target currency (e.g., "JPY").

    Returns:
        The exchange rate information as a json response,
        or None if the rate could not be fetched.
    """
    base_url = "https://hexarate.paikama.co/api/rates/latest"
    api_url = f"{base_url}/{base}?target={target}"

    response = requests.get(api_url)
    if response.status_code == 200:
        return response.json()
```

> **Note:** Pay close attention to the descriptive text within the triple quotes (`"""..."""`). This is called a docstring. While it's good practice for human developers, for an AI Agent it is essential.
>
> The agent's underlying LLM reads the function name and this docstring to understand what the tool does, what its arguments (`base` and `target` in this case) mean, and when it should be used.
>
> A clear, descriptive docstring is the single most important factor for the agent to use your tool correctly.

Now, edit the `agent.py` file: import `get_fx_rate` function and assign it as a `FunctionTool`.

### Update `agent.py` file for Currency Exchange

Copy this codeblock and replace the existing content of `agent.py` file:

```python
from google.adk.agents import Agent
from google.adk.tools import FunctionTool

from .custom_functions import get_fx_rate

root_agent = Agent(
    model='gemini-2.5-flash',
    name='root_agent',
    description='A helpful assistant for user questions.',
    instruction='Answer user questions to the best of your knowledge',
    tools=[FunctionTool(get_fx_rate)]
)
```

> **Note:** Relative import is needed to import the functions from other Python files because ADK runs the agent folder as a Python module. This approach is fine for a small set of tools, but as the tool grows, we will need to create a tool package.

After the changes, start the agent again by typing:

```bash
adk web
```

When the agent is up, ask the same question again, "What is the exchange rate from Singapore dollars to Japanese yen?"

This time around you should see the actual exchange rate given by the `get_fx_rate` tool.

![Currency Exchange Working](/assets/images/12.png "Currency Exchange Working")

Feel free to ask any currency exchange related questions as you wish.

## Integrate with Built-in Google Search Tool

With the agent now capable of providing exchange rates, the next task is to obtain next month's weather forecast. Ask this question to the agent, "What is the weather forecast in Tokyo, Japan for next month?"

![Weather Forecast Query](/assets/images/13.png "Weather Forecast Query")

As you might expect, the weather forecast requires real-time information which our agent doesn't have. While we could code new Python functions for each use case that requires real-time data, adding more and more custom tools quickly makes the agent too complicated and difficult to manage.

Fortunately, the Agent Development Kit (ADK) provides a suite of Built-in Tools, including Google Search that are ready to use, simplifying how our agent interacts with the outside world.

To equip the agent with Google Search tool, you need to implement a multi-agent pattern. First, you create a specialized agent whose sole job is to perform Google searches. Then, you assign this new Google Search Agent to our main `personal_assistant` as a tool. Here are the steps:

### Create `custom_agents.py` file

> **Note:** Before proceeding, make sure you're in the correct directory (`ai-agents-adk`). Run the `pwd` (print working directory) command in your terminal to confirm your current location.

Now, create a Python file named `custom_agents.py` in the `personal_assistant` folder by executing this command in the terminal:

```bash
touch personal_assistant/custom_agents.py
```

This is how your folder structure should look like now:

```text
ai-agents-adk/
└── personal_assistant/
    ├── .env
    ├── __init__.py
    ├── agent.py
    ├── custom_functions.py
    └── custom_agents.py
```

This file will contain the code for the specialized `google_search_agent`. Copy the following code to the `custom_agents.py` file using the Code Editor:

```python
from google.adk.agents import Agent
from google.adk.tools import google_search

# Create an agent with google search tool as a search specialist
google_search_agent = Agent(
    model='gemini-2.5-flash',
    name='google_search_agent',
    description='A search agent that uses google search to get latest information about current events, weather, or business hours.',
    instruction='Use google search to answer user questions about real-time, logistical information.',
    tools=[google_search],
)
```

Once the file is created, update the `agent.py` file as shown below.

### Update `agent.py` file for Google Search Integration

Copy this codeblock and replace the existing content of `agent.py` file:

```python
from google.adk.agents import Agent
from google.adk.tools import FunctionTool
from google.adk.tools.agent_tool import AgentTool

from .custom_functions import get_fx_rate
from .custom_agents import google_search_agent

root_agent = Agent(
    model='gemini-2.5-flash',
    name='root_agent',
    description='A helpful assistant for user questions.',
    tools=[
        FunctionTool(get_fx_rate), 
        AgentTool(agent=google_search_agent),
    ]
)
```

Let's break down the powerful new pattern in the code:

* **A New Specialist Agent**: We've defined a completely new agent, `google_search_agent`. Note its specific description and that its only tool is `google_search`. It's a search specialist.
* **`agent_tool.AgentTool`**: This is a special wrapper from the ADK. It takes an entire agent (our `google_search_agent`) and packages it to look and act like a standard tool.
* **A Smarter `root_agent`**: Our `root_agent` now has a new tool: `AgentTool(agent=google_search_agent)`. It doesn't know how to search the web, but it knows it has a tool it can delegate search tasks to.

Notice the instruction field is gone from `root_agent`. Its instructions are now implicitly defined by the tools it has available.

The `root_agent` has become an orchestrator or a router, whose main job is to understand a user's request and pass it to the correct tool, either the `get_fx_rate` function or the `google_search_agent`. This decentralized design is key to building complex, maintainable agent systems.

Now, type:

```bash
adk web
```

in the terminal to start the instance and ask this question to the agent again, "What is the weather forecast in Tokyo, Japan for next month?"

![Weather Search Working](/assets/images/14.png "Weather Search Working")

The agent is now using `google_search_agent` to get latest information.

You can try asking a current exchange question too. The agent should now be able to use the right tool for the respective question.

![Multiple Tools Working](/assets/images/15.png "Multiple Tools Working")

Feel free to ask other questions that require real-time information to the agent and observe how it handles the queries using the tools at its disposal.

## Leverage LangChain's Wikipedia Tool

Our agent is shaping up to be a great travel assistant. It can handle currency exchange with its `get_fx_rate` tool and manage logistics with its `google_search_agent` tool. But a great trip isn't just about logistics; it's about understanding the culture and history of your destination.

While the `google_search_agent` can find cultural and historical facts, information from a dedicated source like Wikipedia is often more structured and reliable.

Fortunately, ADK is designed to be highly extensible, allowing you to seamlessly integrate tools from other AI Agent frameworks like CrewAI and LangChain. This interoperability is crucial because it allows for faster development time and allows you to reuse existing tools. For this use case, we will leverage the Wikipedia tools from LangChain.

First, stop the running agent process (**Ctrl + C**) and install additional libraries to the current Python virtual environment by typing the following commands in the Terminal:

```bash
uv pip install langchain-community wikipedia
```

### Create `third_party_tools.py` file

> **Note:** Before proceeding, make sure you're in the correct directory (`ai-agents-adk`). Run the `pwd` (print working directory) command in your terminal to confirm your current location.

Now, create a Python file named `third_party_tools.py` in the `personal_assistant` folder by executing the following command in the terminal:

```bash
touch personal_assistant/third_party_tools.py
```

This is how your folder structure should look like now:

```text
ai-agents-adk/
└── personal_assistant/
    ├── .env
    ├── __init__.py
    ├── agent.py
    ├── custom_functions.py
    ├── custom_agents.py
    └── third_party_tools.py
```

This file will contain the implementation for LangChain Wikipedia tool. Copy the following code into the `third_party_tools.py` using the Cloud Editor:

```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# Configure the Wikipedia LangChain tool to act as our cultural guide
langchain_wikipedia_tool = WikipediaQueryRun(
    api_wrapper=WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=3000)
)

# Give the tool a more specific description for our agent
langchain_wikipedia_tool.description = (
    "Provides deep historical and cultural information on landmarks, concepts, and places."
    "Use this for 'tell me about' or 'what is the history of' type questions."
)
```

### Update `agent.py` file for Wikipedia Integration

Now, update the `agent.py` file with the content below:

```python
from google.adk.agents import Agent
from google.adk.tools import FunctionTool
from google.adk.tools.agent_tool import AgentTool
from google.adk.tools.langchain_tool import LangchainTool

from .custom_functions import get_fx_rate
from .custom_agents import google_search_agent
from .third_party_tools import langchain_wikipedia_tool

root_agent = Agent(
    model='gemini-2.5-flash',
    name='root_agent',
    description='A helpful assistant for user questions.',
    tools=[
        FunctionTool(get_fx_rate), 
        AgentTool(agent=google_search_agent),
        LangchainTool(langchain_wikipedia_tool),
    ]
)
```

Now, type:

```bash
adk web
```

in the terminal to start the instance and ask this question to the agent, "Tell me about the history of Kyoto".

![Wikipedia Tool Working](/assets/images/16.png "Wikipedia Tool Working")

The agent correctly identifies this as a historical query and uses its new Wikipedia tool. By integrating a third-party tool and giving it a specific role, you've made your agent significantly more intelligent and useful for its travel-planning purpose.

To see exactly how the agent made this choice, you can use the event inspector in the `adk web` UI. Click on the **Events** tab and then on the most recent `functionCall` event.

![Event Inspector](/assets/images/17.png "Event Inspector")

The inspector shows a list of all available tools and highlights the tool_code for the one that was executed by the agent.

![Tool Selection Details](/assets/images/18.png "Tool Selection Details")

## Clean Up (Optional)

Since this workshop doesn't involve any long-running products, simply stopping your active agent sessions (e.g., the `adk web` instance in your terminal) by pressing **Ctrl + C** in the terminal is sufficient.

### Delete Agent Project Folders and Files

If you only want to remove the code from your Cloud Shell environment, use the following commands:

```bash
cd ~
rm -rf ai-agents-adk
```

### Disable Vertex AI API

To disable the Vertex AI API that was enabled earlier, run this command:

```bash
gcloud services disable aiplatform.googleapis.com
```

### Shut Down the Entire Google Cloud Project

If you wish to fully shut down your Google Cloud project, refer to the [official guide](https://cloud.google.com/resource-manager/docs/creating-managing-projects#shut_down_a_project) for detailed instructions.

## Conclusion

Congratulations! You've successfully empowered the personal assistant agent with custom functions and real-time Google Search access. Read this official documentation on [using tools with Google ADK](https://google.github.io/adk-docs/tools/).

More importantly, you have learned the fundamental architectural pattern for building capable agents: using specialized agents as tools. By creating a dedicated `google_search_agent` and giving it to your `root_agent`, you've taken your first step from building a single agent to orchestrating a simple, yet powerful, multi-agent system.

## Additional Resources

* [Google Codelabs](https://codelabs.developers.google.com/)

* [Agent Development Kit Documentation](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-builder/introduction)

* [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
