#### Message User in telegram

  1. Get TELEGRAM_BOT_TOKEN and TELEGRAM_USER_ID from .env
    - use run_bash_command tool to run command: `cat .env` to read .env file
    - parse TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID from .env 
    

  2. Sending message to user in telegram
    - send a post request to endpoint: "https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage" with payload like below
        
        
    ```
        {
            'chat_id': TELEGRAM_USER_ID,
            'text': <message>
        }
    ```
    

> note: Replace TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID in above template with actual user token and ids from .env


####  Check new LLM papers on arXiv
    - Send GET request to arXiv API
        Example
        
        `GET: https://export.arxiv.org/api/query?search_query=all:llm&start=0&max_results=3&sortBy=submittedDate&sortOrder=descending`
    
    - Parse and prepare results

## Tasks
Tasks file: TASKS.md
when user asks to update tasks: append to TASKS.md using bash command `echo "Task" >> TASKS.md`
to read from tasks: use: `cat TASKS.md`
Send result of task execution to telegram, without asking for user confirmation.