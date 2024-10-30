# FishingBot - Telegram Bot with Azure Functions

**FishingBot** is a Telegram bot designed to answer user questions about fishing by leveraging Azure AI services, including **Custom Question Answering** and **OpenAI GPT models**. The bot is deployed on **Azure Functions** for serverless operation and integrates **Azure Key Vault** for secure access to API keys and other sensitive information. This setup allows FishingBot to respond to fishing-related queries and provide information on various fishing topics.

---

## Features

- **Serverless Architecture**: Runs on Azure Functions for scalability and reduced infrastructure maintenance.
- **Azure Key Vault Integration**: Securely manages and retrieves API keys.
- **Custom Question Answering and OpenAI GPT-4**: Uses AI to provide relevant, conversational responses to fishing-related questions.
- **Telegram Integration**: Operates through Telegram, handling user queries in real-time.
- **Rate Limiting**: Throttles requests to avoid hitting rate limits for external APIs.
- **Contextual Replies**: Handles multi-turn conversations by preserving context in replies.

## Training the Bot
You can add new entries to the CQA knowledge base through the bot:

### 1. Update Knowledgebase Command
To train the bot, you can send a command like:

```bash
/update_kb "What is fly fishing?" "Fly fishing is an angling method that uses a light-weight lure called an artificial fly."

```go
qna_pairs := []QnA{
    {"question": "What is euro nymphing?", "answer": "Euro nymphing is a short-line nymphing method that uses thin leaders and weighted flies."},
}
updateCqaKnowledgebase(qna_pairs)

```
This function will update the knowledge base by making a PATCH request to the CQA endpoint.

---

## Folder Structure
```
/FishingBotFunction
├── main.go                   # Main entry point for the Azure Function
├── go.mod                    # Go module dependencies
├── go.sum                    # Go module checksums
├── api_requests.go           # Handles API requests to CQA and OpenAI
├── secrets_manager.go        # Manages Key Vault interactions for secure secrets access
├── telegram_handler.go       # Processes Telegram messages, manages context and throttling
└── .gitignore                # Specifies files to ignore in Git repository

```
---
## File Descriptions

### `main.go`
The entry point for the Azure Function, responsible for receiving HTTP triggers from Telegram. This file processes incoming messages, retrieves the necessary secrets, and delegates the processing to `telegram_handler.go`.

- **Function**: `main`
- **Purpose**: Responds to HTTP triggers, loads secrets, and forwards the message to the Telegram handler.
- **Dependencies**: `secrets_manager.go`, `telegram_handler.go`

### `api_requests.go`
This module handles requests to external AI services, specifically **Custom Question Answering (CQA)** and **OpenAI**. It includes rate-limiting to prevent hitting API request limits and utilizes asynchronous requests for efficient, non-blocking I/O operations.

- **Functionality**:
  - `queryCQA`: Queries Azure's Custom Question Answering service to retrieve answers based on user questions.
  - `queryOpenAI`: Sends a prompt to the OpenAI GPT model to generate conversational responses.
  - **Rate Limiting**: Uses `asyncio_throttle.Throttler` to manage the frequency of API calls.
- **Usage**: Called from `telegram_handler.go` to get responses for user questions.

### `secrets_manager.go`
Manages interaction with **Azure Key Vault** for securely storing and retrieving sensitive information like API keys. By isolating secret management, this module keeps secrets secure and enables easy updates without changing the rest of the code.

- **Functionality**:
  - `loadSecrets`: Connects to Azure Key Vault using the Function App’s Managed Identity to fetch secrets like `TELEGRAM_API_TOKEN`, `CQA_ENDPOINT`, `CQA_API_KEY`, `OPENAI_ENDPOINT`, and `OPENAI_API_KEY`.
- **Usage**: Called in `main.go` to load and pass secrets securely to other modules.

### `telegram_handler.go`
This module handles the core processing of Telegram messages. It manages context for multi-turn conversations, applies throttling, and handles the decision-making on whether a response is warranted based on the type of message received.

- **Functionality**:
  - `handleTelegramMessage`: Parses incoming messages, manages user context, throttles requests, and calls `api_requests.go` functions to generate responses.
  - **Context Management**: Maintains a simple history to handle follow-up questions from users.
  - **Throttling**: Ensures rate limits are adhered to when querying external APIs.
- **Usage**: Invoked by `main.go` when a new Telegram message arrives.

### `go.mod` and `go.sum`
These files manage dependencies for the Go application, ensuring all necessary packages are included during deployment.

- **Usage**: Ensures the application installs all necessary packages during deployment.

### `.gitignore`
Defines files and directories to exclude from the Git repository, similar to a `.funcignore` file. Helps keep the repository clean by excluding unnecessary files.

- **Example Entries**: `.env`, `*.exe`, etc.

---

## Deployment Instructions

1. **Configure Azure Key Vault**: Ensure all necessary secrets are stored in Azure Key Vault and accessible by the Function App’s Managed Identity.
2. **Set up Azure Storage Account**: Assign the `AzureWebJobsStorage` setting with your storage account’s connection string.
3. **Deploy to Azure**:
   - Use GitHub Actions or Azure CLI (`func azure functionapp publish <FunctionAppName>`) to deploy the Function App.
4. **Set Telegram Webhook**: Update the Telegram bot webhook to point to your Function App URL.

---

## Usage

- **Send Messages**: Users can interact with FishingBot via Telegram by sending questions or following up on previous answers.
- **Bot Behavior**: FishingBot will process questions asynchronously, retrieve responses from CQA or OpenAI, and reply with relevant information about fishing.

---

## Example Usage

- **User**: “What’s the best fly to use in Florida waters?”
- **Bot**: “The Clouser Minnow is popular in Florida’s saltwater flats. Let me know if you’d like more information!”

---

FishingBot is now ready to serve fishing enthusiasts on Telegram, providing information powered by Azure AI! Feel free to customize it further or add additional services as needed.