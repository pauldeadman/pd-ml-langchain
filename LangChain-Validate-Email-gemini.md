# Simple Tool to Send Email with Gemini 2.5 Flash Lite

## Create Basic Tool

```
from langchain.tools import tool

@tool
def send_email(
    to: list[str],  # email addresses
    subject: str,
    body: str,
    cc: list[str] = []
) -> str:
    """Send an email via email API. Requires properly formatted addresses."""
    # Stub: In practice, this would call SendGrid, Gmail API, etc.
    # print(f"DEBUG Email sent to {', '.join(to)} - Subject: {subject}")
    return f"Email sent to {', '.join(to)} - Subject: {subject}"
```

## Create Agent

```
from langchain.tools import tool

@tool
def send_email(
    to: list[str],  # email addresses
    subject: str,
    body: str,
    cc: list[str] = []
) -> str:
    """Send an email via email API. Requires properly formatted addresses."""
    # Stub: In practice, this would call SendGrid, Gmail API, etc.
    # print(f"DEBUG Email sent to {', '.join(to)} - Subject: {subject}")
    return f"Email sent to {', '.join(to)} - Subject: {subject}"
```

## Create LLM

```
import os
from langchain_google_genai import ChatGoogleGenerativeAI

os.environ["GOOGLE_API_KEY"] = "AQ......."

model = ChatGoogleGenerativeAI(model="gemini-2.5-flash-lite")
```

## Call LLM

```
query = "Send email to paul@pauldeadman.com an urgent reminder about reviewing the new application design mockups"

for step in email_agent.stream(
    {"messages": [{"role": "user", "content": query}]}
):
    for update in step.values():
        for message in update.get("messages", []):
            message.pretty_print()
            
```

## Full Example

[LanChain Agent Send Email](https://github.com/pauldeadman/pd-ml-langchain/blob/main/LangChain-Validate-Email-gemini-pub.ipynb)
