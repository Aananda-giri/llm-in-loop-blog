# LLMs In Loop
- <tagline> Build Chatbot, Coding CLI and openclaw from scratch

We will cover:

1. How to do Inference and and what are tool use
2. Chatbot: with context (from scratch)
3. Coding CLI: `Claude CLI` (from scratch)
4. Digital Assistant: minimal openClaw alternative (from Scratch) 

Note: All the code in this page is pseudocode, you can find actual notebook accompanying this blog at: [walkthrough.ipynb](./walkthrough.ipynb)

## 1. Introduction
LLMs have been with us for long time and over time, the different applications have been built around them with different functions based on domain of work. In this blog post, I would like to argue that all these applications are simply LLM in loop, with domain specific tools interacting with an environment.

### 1.1 LLM Inference:
- Inference is term that refers to asking LLM to do generate next token and Prompts are instructions we give LLM to get the desired output (e.g. asking chatgpt, what is ...)

In this blog-post, we are using Gemini as our LLM provider, and `gemini-2.5` as our model of choice, we can always use more powerful models like `gemini-3` or `claude-4.6` for better results.

- Note: Before we proceed, get your api key from [Google AI Studio](https://aistudio.google.com/app/api-keys) and place it in `.env` file

we will use function like below to perform inference throughout rest of the sections, please refer to [walkthrough.ipynb](walkthrough.ipynb) for actual code

```
def respond(query):
    response = <infenrence-code: Get response for query>
    return response
```

## 1.2 The Loop (Birth of Chatbot)

We defined `generate` function in previous section, lets put it in a loop and we will get a chatbot (like chatgpt).

```
while True:
    query = input("User")
    response = respond(query)
    print(response)

'''
## Output
User: What are the must-see places in Pokhara?
LLM: Phewa Lake, World Peace Pagoda, Sarangkot, ...
'''
```

Lets try asking it a follow up question
```
User: What are the must-see places in Pokhara?
LLM: Phewa Lake, World Peace Pagoda, Sarangkot, ...

User: I prefer peaceful spots - can you adjust the plan?
LLM: ... where you're planning to go?
```


It does not know previous question because we are only giving it the current query, lets store it in variable context

### 1.3 Conversationi History
We will store our conversation with LLM in list named `history` and pass history to `respond` function
 
```
history = []

while True:
    query = input("User")
    history.append(query)
    
    response = respond(query)
    history.append(response)

    print(response)

'''
## Output
User: What are the must-see places in Pokhara?
LLM: Phewa Lake, World Peace Pagoda, Sarangkot, ...

User: I prefer peaceful spots - can you adjust the plan?
LLM: For peaceful spots in Pokhara, consider visiting World Peace Pagoda.
'''
```

Now, it can answer follow-up questions.



### 1.4 Tools (Agency Appears):
- Tools are functions that a LLM can execute with custom parameters. LLMs can only genereate tokens and tool calling is using those tokens to execute functions. Tools are like hands and legs of LLMs, which help them to perform tasks in real world, like searching web, installing dependencies, writing or executing code and much more. It also expands knowledge of LLM beyond the data it was trained on, so that they dont have to memorise everythihng.

```
# Weather tool

# 1. Tool Implementation
def get_weather(place_name):
    weather = <get-weather-via-some-API-call>
    return weather
```

```
# 2. respond function
def respond(query, config):
    response = <infenrence-code: Execute tool or Get response>
    return response
```


Lets write a very basic system prompt. System prompt is high-level, foundational instructions given to define its persona and behaviour throughout the conversation. They are useful to set the tone, boundaries and output format of the model.

```
# 3. Config
config = GenerateContentConfig(
    tools=[get_weather],
    system_instruction="You are a general-purpose helpful assistant..."
)
```

```
# 4. The Loop

history = []

while True:
    query = input("User")
    history.append(query)
    
    response = respond(query, config=config)
    history.append(response)

    print(response)
'''
## Response
User:  What is the weather in Pokhara right now?
LLM: <Checking weather for 'Pokhara'...>
The weather in Pokhara is rainy
'''
```

- I have omitted tool defination part here (which is organizing the tool in proper format for LLM), please go through [walkthrough.ipynb](./walkthrough.ipynb) for complete implementation.

- Also refer to [gemini-api-docs](https://ai.google.dev/gemini-api/docs/function-calling) for more on function calling


## 2. Coding Agent: `Minimal Claude CLI implementation (from scratch)`
Coding is one of the intellectual fields we have been hoping to automate long before the advent of ChatGPT. There was github copilot (before ChatGpt), that used to do autocompletes, since the advent of ChatGPT, we have been getting increasingly better coding assistants simply by putting the LLM in loop and askinig it to generate the code (with some clever techniques like giving LLM only certain part of the code). Some examples of Agentic coding editors are: Cursor, openai-codex, Claude Code, Gemini CLI, Qwen CLI, ...

In this section, we will create a minimal coding CLI from scratch just three tools: Read tool, Write tool and List tool and we will put LLM (with tools) in loop.

Lets first implementn the tools:
```

# 1. Tool implementations

def read_file(path, offset=0, limit=None):
    """Read file with line numbers.""" 
    content = read(path)
    return content[offset:limit]

def write_file(path, content):
    """Write content to file."""
    with open(path, 'w') as f:
        f.write(content)
    return "ok"

def list(path):
    """List files and directories"""

    results = <full-path-of-files-in-path>
    return results
```


```
# 2. respond function
def respond(query, config):
    response = <infenrence-code: Execute tool or Get response>
    return response
```

```
# 3. config
config = GenerateContentConfig(
    tools=[read_files, write_files, list],
    system_instruction=f"Concise coding assistant. cwd: {os.getcwd()}"
)

```

Finally, lets put it in loop
```
# 4. The Loop

history = []

while True:
    query = input("User")
    history.append(query)
    
    response = respond(query, config=config)
    history.append(response)

    print(response)
```

Example Interaction
```
User: create file add.py with function to add two numbers
LLM: File add.py created with function to add two numbers.

User: create file substract.py with function to substract two numbers
LLM: File substract.py created with function to substract two numbers.

User: create file math.py that imports these two functions and perform demo calculation
LLM: File math.py created. It imports `add` and `substract` functions and performs demo calculations, printing the results.

```

It has generated following files:
```
# add.py
def add(a, b):
    return a + b

# substract.py
def substract(a, b):
    return a - b


# math.py
from add import add
from substract import substract

result_add = add(5, 3)
print(f"5 + 3 = {result_add}")

result_substract = substract(10, 4)
print(f"10 - 4 = {result_substract}")
```

- refer to [nanocode](https://github.com/1rgs/nanocode/blob/master/nanocode.py), ~250 line, Minimal Claude Code alternative for more minimal tools implementation.

One of the very powerful tool which would be very helpful in codinig tasks is: `run_bash_command tool`. It is a tool that can run bash command and perform tasks like running grep commands, compiling generated code, installing dependencies and much more. we will implement it in next section.



## 3. Autonomous Personal Assistant: `Minimal openClaw Implementation - from Scratch` 
Openclaw (formerly clawdbot) is the fastest growing Agentic personal assistant.

In this section, we will try to create minimal version of openclaw and ask it to search arxiv for new llm papers and send them to me via telegram.


### 3.1 SKILLS.md
SKILLS.md contains instructions which LLM can use to perform tasks (similar to tool calling)

We can search new llm papers from arxiv with GET request tourl like: `https://export.arxiv.org/api/query?search_query=all:llm&start=0&max_results=3&sortBy=submittedDate&sortOrder=descending` 

and to send message to telegram, we have to ping `https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage` with payload like below:
```
{
    'chat_id': TELEGRAM_USER_ID,
    'text': <message>
}
```

Lets start by placing these information in `SKILLS.md`
```
# SKILLS

## Message User in telegram

1. Get Telegram token and user_id
use run_bash_command to get TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID from .env file, use bash command: `cat .env` to read them from .env file

2. Sending message
send a post request to endpoint: "https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage" with payload like below
    

{
    'chat_id': TELEGRAM_USER_ID,
    'text': <message>
}


## Checking new research papers on LLM
send a GET request to `export.arxiv.org` e.g.
https://export.arxiv.org/api/query?search_query=all:llm&start=0&max_results=3&sortBy=submittedDate&sortOrder=descending

```

Note: For sending messages to user in telegram, we will need `TELEGRAM_BOT_TOKEN` and `user_id`
  - To get `TELEGRAM_BOT_TOKEN`, chat with @BotFather in telegram. type `/newbot` or `/mybots` and proceed untill you get token
  - To get `user_id`, send message: `/start` to @userinfobot in telegram.



### 3.2 Tool: `run_bash_command`
Instead of defining seperate tools for these tasks (sending telegram msg., arxiv search), we will write single tool: `run_bash_command` and ask LLM to run commands based on instructions from `SKILLS.md`.

Here is what the tool implementation look like:

```
# 1. Tool Implementation
def run_bash_command(command):
    """Execute a bash command and return output."""
    try:
        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True
        )
        return result.stdout if result.stdout else result.stderr
    
    except Exception as err:
        return f"Error: {err}"
```



This is our function from previous section with a minor modification, we have added `max_iteration` to allow multiple bash command executions it responds to user.

```
# 2. respond function
def respond(history):
    max_iterations = 10

    for iteration in range(max_iterations):
        response = <infenrence-code: Execute tool or Get response>

        
        if response.tool_call:
            result = execute_function
            history.append(result)
        else:
            return result.text

```

```
# 3. Config
config = GenerateContentConfig(
    tools=[run_bash_command],
    system_instruction="You are a helpful assistant that can execute bash commands..."
)
```

And this is the loop (same as previous section):
```
# 4. The Loop

history = []

while True:
    query = input("User")
    history.append(query)
    
    response = respond(query, config=config)
    history.append(response)

    print(response)
```




Example Interaction
```
User: find research papers on llm from arxiv and send them to me on telegram
LLM: reads SKILLS.md -> reads TASKS.md -> reads TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID -> fetches new papers -> sends them to telegram
```



### 3.1 Heartbeat
Heartbeat is what keeps the agent alive. It has cycle with following components: 

1. Sleep: The agent is put to sleep when idle to save tokens and compute
2. Wake-Up: The agent wakes after certain time, in something like HEARTBEAT_INTERVAL
3. The Check-and-Act: It checks if there are any tasks to be performed and perform those tasks, reports to user if necessary and goes back to sleep. Task may include checking application logs periodically to find any bugs, and report to user if there are critical bugs.


#### 3.1.1 TASKS.md
lets write `TASKS.md` to define things the agent should do when it wakes up.

```

find research papers on llm from arxiv and send them to me on telegram
```

Here is the loop with heartbeat example
```
history = []

while True:
    query = "Read TASKS.md, and execute the instructions"
    
    history.append(query)

    response = respond(query, config)
    history.append(response)

    # Note: This is the Only thing we are adding in previous code
    print("sleeping...")
    time.sleep(HEARTBEAT_INTERVAL)
```

Example Flow
```
LLM: Agent wakes up -> reads SKILLS.md -> reads TASKS.md -> reads TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID -> fetches new papers -> sends them to telegram -> goes back to sleep
```

#### 3.1.2 Non-Blocking Loop
That works but TASKS.md is hardcoded and the loop is blocking so user can't give it input when the agent is sleeping.

We will use a background thread for heartbeat loop so that main loop can still react to user input.

and In order to avoid hardcoded TASKS.md, We can ask it to update TASKS.md in main thread and heartbeat

```
- Main loop → reactive user input
- Background thread → periodic tasks (heartbeat)
- Both run concurrently, independent of each other
```

In our toy example, we can run another loop in background that runs once every `HEARTBEAT_INTERVAL`.

```
import threading
import time

# Background Loop for TASKS.md
# -----------------------------
def heartbeat_loop(HEARTBEAT_INTERVAL):
    while True:
        query = "Read TASKS.md, and execute the instructions"
        response = respond(query, config)
        time.sleep(HEARTBEAT_INTERVAL)

# Run heartbeat in background
threading.Thread(target=heartbeat_loop, args=(86400,), daemon=True).start()



# Main reactive loop
# -------------------
history = []
while True:
    query = "Read TASKS.md, and execute the instructions"
    
    history.append(query)

    response = respond(query, config)
    history.append(response)

    # Note: This is the Only thing we are adding in previous code
    print("sleeping...")
    time.sleep(HEARTBEAT_INTERVAL)
```

Example Flow
```
User: Update TASKS.md with: find research papers on llm from arxiv and send them to me on telegram
LLM: updates TASKS.md

LLM: wakes up (after) -> reads SKILLS.md -> reads TASKS.md -> reads TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID -> fetches new papers -> sends them to telegram -> goes back to sleep
```

update task: get research papers from arxiv and send them to me on telegram
read TASKS.md, SKILLS.md and complete tasks

Add whatsapp, some other tools and run it 24/7 and it becomes openClaw (crab emoji)
https://github.com/openclaw/openclaw
