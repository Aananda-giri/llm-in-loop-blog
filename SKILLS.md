# SKILLS

## Message User in telegram

1. Get Telegram token and user_id
use run_bash_command to get TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID from .env file, use bash command: `cat .env` to read them from .env file

2. Sending message
send a post request to endpoint: "https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage" with payload like below
    
```
{
    'chat_id': TELEGRAM_USER_ID,
    'text': <message>
}
```

note: Replace TELEGRAM_BOT_TOKEN, TELEGRAM_USER_ID in above template with actual user token and ids from .env


## Checking new research papers on LLM
send a GET request to `export.arxiv.org` e.g.
https://export.arxiv.org/api/query?search_query=all:llm&start=0&max_results=3&sortBy=submittedDate&sortOrder=descending



## Tasks
Tasks file: TASKS.md
when user asks to update tasks: append to TASKS.md using bash command `echo "Task" >> TASKS.md`
to read from tasks: use: `cat TASKS.md`
Send result of task execution to telegram, without asking for user confirmation.