# Chat
ChatGPT-based chat application.

## Table of Contents:
- [About](#about);
- [Initialization](#initialization);
- [Environment](#environment);
- [Routes](#routes).

## About
ChatGPT-based Solid.js chat application with per-user token usage control and a Pipedrive integration.

## Initialization
For a local deployment, make sure to include required [environment variables](#environment).

Building the project (/app/):
```sh
vite build
```

Initializing the app (/server/):
```sh
node main.js
```

## Environment
Environment variables used by the project.
- `ADDRESS`: Bind address. 
    - Required.
    - Usually `127.0.0.1`.
- `PORT`: Bind port. 
    - Required.
    - Usually `3000`. 
- `DB_URL`: Database Url. 
    - Required.
    - URL for the MongoDB database, usually `mongodb://localhost:27017`.
- `DIST_URL`: Distribution URL.
    - Required.
    - Location on which the chat will be running.
- `LOGIN_URL`: Login URL.
    - Required.
    - Location on which the user will be redirected in case of invalid credentials.
- `INT_URL`: Integration URL. 
    - Required.
    - Location for the main integration.
- `SHARED_SECRET`: Shared Secret. 
    - Required.
    - Token used for JWT signature verification.
- `OAI_ASSISTANT`: OpenAI Assistant. 
    - Required.
    - ID for the OpenAI Assistant on which the Chat will be based upon.
- `OAI_TOKEN`: OpenAI Token.
    - Required.
    - OpenAI API Token.
- `PIPEDRIVE_TOKEN`: Pipedrive Token. 
    - Required.
    - Pipedrive API Token.

## Routes
Project routes, can be split into two main routes:
- Chat: `/server/routes/chat/index.js`;
- Dashboard: `/server/routes/dashboard/index.js`.

They're both imported by the main server file, `/server/main.js`.

![](docs/static/routes.svg)

### /chat/thread

**GET** `/chat/thread`

**Description:**  
Creates a new chat thread or retrieves an existing one based on the session. If the user doesn't exist, it will create a new user with default token values. If the user's token balance is depleted, it will grant new tokens based on the token grant settings.

**Query Parameters:**
- `s` (string): The session identifier.

**Responses:**
- **200 OK**: Returns the thread details along with user information and session data.
- **401 Unauthorized**: If the session is invalid or not provided.
- **500 Internal Server Error**: If an error occurs during the request.

### /chat/file

**GET** `/chat/file`

**Description:**  
Retrieves a file associated with a chat thread based on the provided file ID.

**Query Parameters:**
- `id` (string): The ID of the file to retrieve.

**Responses:**
- **200 OK**: Returns the file content.
- **400 Bad Request**: If the file ID is not provided or is invalid.

### /chat/flag

**GET** `/chat/flag`

**Description:**  
Flags a specific message in a chat thread. Requires a valid session and specific message details.

**Query Parameters:**
- `t` (string): The thread ID.
- `m` (string): The message ID.
- `s` (string): The session identifier.

**Responses:**
- **200 OK**: Indicates that the message has been successfully flagged.
- **400 Bad Request**: If required query parameters are missing.
- **401 Unauthorized**: If the session is invalid or does not match.
- **500 Internal Server Error**: If an error occurs during the request.

### /chat/message

**GET** `/chat/message`

**Description:**  
Sends a message to a chat thread and processes it. Handles token usage and updates user tokens accordingly. Streams the response back as events.

**Query Parameters:**
- `t` (string): The thread ID.
- `m` (string): The message content.
- `s` (string): The session identifier.

**Responses:**
- **200 OK**: Streams the response events including message deltas and usage information.
- **400 Bad Request**: If required query parameters are missing.
- **401 Unauthorized**: If the session is invalid or does not match.
- **429 Too Many Requests**: If the user's token balance is insufficient.
- **500 Internal Server Error**: If an error occurs during the request.

### /dashboard/overall/usage

**GET** `/dashboard/overall/usage`

**Description:**  
Retrieves overall usage statistics for a given date range, including tokens used and average usage. Provides detailed breakdowns by day, month, company, and user.

**Query Parameters:**
- `date_from` (string): The start date for the usage statistics (ISO 8601 format).
- `date_to` (string): The end date for the usage statistics (ISO 8601 format).

**Responses:**
- **200 OK**: Returns the usage statistics and breakdowns.
- **400 Bad Request**: If date parameters are invalid.
- **500 Internal Server Error**: If an error occurs during the request.

### /dashboard/overall/resume

**POST** `/dashboard/overall/resume`

**Description:**  
Generates a summary of the usage statistics based on the provided data. Utilizes a GPT model to create a narrative overview of the usage information.

**Request Body:**
- `stats` (object): The usage statistics data.

**Responses:**
- **200 OK**: Returns the generated summary.
- **500 Internal Server Error**: If an error occurs during the request.

### /dashboard/transactions

**GET** `/dashboard/transactions`

**Description:**  
Retrieves a list of transactions with pagination and filtering options. Includes aggregate statistics for the specified date range.

**Query Parameters:**
- `limit` (number): The number of transactions to return.
- `start` (number): The number of transactions to skip (for pagination).
- `date_from` (string): The start date for the transactions (ISO 8601 format).
- `date_to` (string): The end date for the transactions (ISO 8601 format).

**Responses:**
- **200 OK**: Returns the transactions and aggregate statistics.
- **400 Bad Request**: If query parameters are invalid.
- **500 Internal Server Error**: If an error occurs during the request.

### /dashboard/users

**GET** `/dashboard/users`

**Description:**  
Retrieves a list of users with pagination options.

**Query Parameters:**
- `limit` (number): The number of users to return.
- `start` (number): The number of users to skip (for pagination).

**Responses:**
- **200 OK**: Returns the list of users.
- **400 Bad Request**: If query parameters are invalid.
- **500 Internal Server Error**: If an error occurs during the request.

### /dashboard/users/:id

**GET** `/dashboard/users/:id`

**Description:**  
Retrieves details of a specific user by their ID.

**Path Parameters:**
- `id` (string): The ID of the user to retrieve.

**Responses:**
- **200 OK**: Returns the user details.
- **500 Internal Server Error**: If an error occurs during the request.

### /dashboard/users/:id/tokens

**POST** `/dashboard/users/:id/tokens`

**Description:**  
Updates the token balance and grant amount for a specific user.

**Path Parameters:**
- `id` (string): The ID of the user to update.

**Request Body:**
- `token_balance` (number): The new token balance.
- `token_grant` (number): The new token grant amount.

**Responses:**
- **200 OK**: Confirms that the token balance and grant amount have been updated.
- **400 Bad Request**: If required parameters are missing or invalid.
- **500 Internal Server Error**: If an error occurs during the request.

### /int/tokens

**POST** `/int/tokens`

**Description:**  
Updates or creates a user with a specified token quota based on the user ID.

**Query Parameters:**
- `id` (string): The internal ID of the user.
- `quota` (number): The token quota to set for the user.

**Request Body:**
- `int` (object): User-specific data.

**Responses:**
- **200 OK**: Confirms that the user’s tokens have been updated.
- **400 Bad Request**: If required parameters are missing or invalid.
- **500 Internal Server Error**: If an error occurs during the request.