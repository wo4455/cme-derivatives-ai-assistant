# CME Derivatives AI Assistant

A full-stack AI learning assistant built to help users explore **CME Group and derivatives-market concepts** through a conversational interface. The project combines a React/Vite frontend with an Express backend, OpenAI API integration, conversation context, user authentication, MongoDB-backed chat persistence, and experiments with OpenAI fine-tuning.

> **Disclaimer:** This is an independent educational project and is not affiliated with, endorsed by, or an official product of CME Group. It does not provide financial advice.

## Overview

The application was designed as a domain-focused chatbot for users who are new to futures, derivatives, and CME Group. Users can enter questions through a React chat interface, and the Express backend sends those questions—along with prior conversation history—to an OpenAI chat model using a CME-focused system prompt.

The repository also contains a complete account system and persistence layer: users can register and log in, passwords are hashed with bcrypt, JWTs are generated for authenticated users, and saved chat histories can be stored in MongoDB through Mongoose.

In addition to the primary chatbot flow, the project contains a separate `FineTuning` workspace with a JSONL training dataset and scripts for uploading training data, creating fine-tuning jobs, listing jobs, checking status, and managing fine-tuned models.

## Key Features

- **Domain-focused AI assistant** for CME Group and derivatives-related questions
- **OpenAI API integration** through the official Node.js SDK
- **Conversation context** passed back to the model across messages
- **React chat interface** with user and AI message rendering
- **Loading states and automatic chat scrolling**
- **User registration and login**
- **bcrypt password hashing**
- **JWT generation** for authenticated users
- **Joi input validation and password-complexity checks**
- **MongoDB / Mongoose user storage**
- **Save and load conversation history** for registered users
- **React Router** login and signup flows
- **React Context** for shared message and loading state
- **OpenAI fine-tuning experiments** using JSONL training data
- **Winston exception logging**
- **Helmet and compression** for production middleware
- **Vite + Tailwind CSS** frontend tooling

## Architecture

```text
cme-derivatives-ai-assistant/
├── client/
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   │   ├── Display.jsx
│   │   │   ├── Header.jsx
│   │   │   ├── Input.jsx
│   │   │   └── Message.jsx
│   │   ├── context/
│   │   │   ├── LoadingContext.jsx
│   │   │   └── MessagesContext.jsx
│   │   ├── routes/
│   │   │   ├── Login.jsx
│   │   │   └── Signup.jsx
│   │   ├── App.jsx
│   │   ├── Root.jsx
│   │   └── main.jsx
│   ├── package.json
│   ├── tailwind.config.js
│   └── vite.config.js
│
└── server/
    ├── FineTuning/
    │   ├── data/
    │   │   └── cme.jsonl
    │   ├── fineTune.js
    │   ├── listFineTunes.js
    │   └── upload.js
    ├── Models/
    │   └── User.js
    ├── config/
    │   ├── custom-environment-variables.json
    │   └── default.json
    ├── middleware/
    ├── routes/
    │   ├── auth.js
    │   ├── chat.js
    │   ├── load.js
    │   ├── main.js
    │   ├── save.js
    │   └── user.js
    ├── startup/
    │   ├── db.js
    │   ├── logging.js
    │   ├── prod.js
    │   └── routes.js
    ├── index.js
    └── package.json
```

## Tech Stack

### Frontend

- React 18
- Vite
- React Router
- React Context
- Tailwind CSS
- React Icons
- JavaScript

### Backend

- Node.js
- Express.js
- OpenAI Node SDK
- MongoDB
- Mongoose

### Authentication & Validation

- bcrypt
- JSON Web Tokens
- Joi
- `joi-password-complexity`
- `node-config`

### Server & Production Middleware

- CORS
- Helmet
- Compression
- Winston
- `express-async-errors`
- dotenv

## AI Request Flow

The main chatbot flow is implemented through the React `Input` component and the Express `/chat` route.

```text
User enters question
        ↓
React Input component
        ↓
POST /chat
        ↓
Express backend
        ↓
System prompt + conversation history + new message
        ↓
OpenAI Chat Completions API
        ↓
AI response
        ↓
React message state
        ↓
Chat display
```

### Conversation Context

The frontend maintains a conversation-history array containing alternating user and assistant messages. Each new request sends the existing history to the backend along with the latest prompt.

The backend constructs a conversation containing:

1. A CME-focused system instruction
2. Previous conversation messages
3. The newest user message

This allows follow-up questions to retain context during the current browser session.

## AI Integration

The backend initializes the OpenAI client with:

```text
API_KEY
```

from the server environment.

The current repository calls the Chat Completions API using the historical model identifier:

```text
gpt-4-0125-preview
```

Because this project was built against an older OpenAI API/model configuration, the model name may need to be updated before running the application today.

### Important Scope Note

The system prompt instructs the assistant to answer questions about CME Group, but the repository does **not** currently implement live web browsing, retrieval-augmented generation (RAG), or direct CME Group website search.

Responses therefore come from the configured OpenAI model plus the conversation context—not from a live CME data source.

## Authentication

The backend includes registration and login routes.

### Registration

`POST /new`

The registration flow:

1. Validates the submitted username and password with Joi.
2. Runs an additional password-complexity check.
3. Checks whether the username already exists.
4. Hashes the password with bcrypt.
5. Stores the new user through Mongoose.
6. Generates a JWT.
7. Returns the token in the `X-Auth-Token` response header.

### Login

`POST /auth`

The login flow:

1. Validates the request.
2. Looks up the username in MongoDB.
3. Compares the submitted password with the stored bcrypt hash.
4. Generates a JWT when credentials are valid.
5. Returns the token in the `X-Auth-Token` response header.

The user model signs JWTs with the configured:

```text
app_jwtPrivateKey
```

environment variable.

## Saved Conversations

The `User` schema includes a `savedChats` array.

### Save

```text
POST /save
```

The server finds the user by username, replaces the user's `savedChats` value with the supplied conversation, and saves the updated record.

### Load

```text
POST /load
```

The server finds the user and returns the stored `savedChats` array.

The React chat display includes UI behavior for saving and loading these conversations.

## Fine-Tuning Experiments

The `server/FineTuning` directory contains an experimental OpenAI fine-tuning workflow.

### Training Data

```text
server/FineTuning/data/cme.jsonl
```

contains JSONL-formatted training data.

### Upload Training Data

`upload.js` creates an OpenAI file with the `fine-tune` purpose.

### Create Fine-Tuning Job

`fineTune.js` demonstrates creating a fine-tuning job against a GPT-3.5-era model.

### Manage Fine-Tuning Jobs

`listFineTunes.js` includes examples for:

- listing fine-tuning jobs
- retrieving job status
- canceling a job
- deleting a fine-tuned model

These scripts are best treated as historical experiments because the OpenAI model IDs and fine-tuning interfaces referenced by the repository may have changed.

## API Routes

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/chat` | Send a prompt and conversation history to the AI backend |
| `POST` | `/new` | Register a user |
| `POST` | `/auth` | Authenticate an existing user |
| `POST` | `/save` | Save a user's conversation history |
| `POST` | `/load` | Retrieve a user's saved conversation history |
| `GET` | `*` | Serve the built React application |

## Running Locally

### Prerequisites

- Node.js
- npm
- An OpenAI API key
- MongoDB if using authentication and saved-chat persistence

### 1. Clone the repository

```bash
git clone https://github.com/wo4455/CME-BOT.git
cd CME-BOT
```

If the repository has been renamed, substitute the new repository name.

### 2. Install frontend dependencies

```bash
cd client
npm install
```

### 3. Install backend dependencies

In a second terminal:

```bash
cd server
npm install
```

### 4. Configure server environment variables

Create:

```text
server/.env
```

with values similar to:

```env
API_KEY=your_openai_api_key
MONGODB_URI=your_mongodb_connection_string
app_jwtPrivateKey=your_jwt_secret
```

Do not commit `.env` or real credentials.

### 5. Enable MongoDB initialization

The repository contains a MongoDB initialization module in:

```text
server/startup/db.js
```

but the `initializeDB()` call is currently commented out in `server/index.js`.

If you want registration, login, and saved-chat persistence to operate against MongoDB, restore the database initialization call before starting the backend.

### 6. Start the backend

From `server/`:

```bash
npm start
```

The Express server uses port `3000` unless `PORT` is configured.

### 7. Start the frontend

From `client/`:

```bash
npm run dev
```

Vite typically serves the development application on:

```text
http://localhost:5173
```

## Current Project Notes

This repository represents a substantial full-stack AI project, but it was built against older OpenAI APIs and contains a few implementation details worth cleaning up before treating it as production-ready:

- The current OpenAI model identifier is historical and may no longer be available.
- The application does not actually browse `cmegroup.com`, despite language in the system prompt.
- MongoDB initialization is currently commented out in the server entry point.
- The registration route's bcrypt salt generation should be reviewed before deployment.
- The client currently hardcodes `http://localhost:3000` for backend requests.
- There is no automated test suite.
- Fine-tuning scripts reference historical model/job identifiers.
- Authentication tokens are returned to the client, but route-level authorization should be strengthened before production use.

## What I Practiced

This project gave me hands-on experience with:

- Integrating an LLM API into a full-stack application
- Designing a domain-specific AI assistant
- Maintaining multi-turn conversation context
- Building React component and Context-based state architecture
- Creating Express API routes
- Designing MongoDB/Mongoose user persistence
- Implementing registration and login flows
- Hashing passwords with bcrypt
- Generating JWT authentication tokens
- Saving and restoring user-generated conversation data
- Using Joi for server-side validation
- Managing secrets with environment variables
- Structuring backend startup, route, model, middleware, and configuration modules
- Experimenting with OpenAI fine-tuning pipelines
- Adding production HTTP middleware and server logging

## Project Positioning

This project is best described as a **full-stack domain-specific AI assistant prototype**. It demonstrates AI integration, frontend/backend communication, user authentication, persistent application state, and early model-customization experimentation.

It should not be described as a live CME data platform, an official CME Group tool, or a production financial-advice system.
