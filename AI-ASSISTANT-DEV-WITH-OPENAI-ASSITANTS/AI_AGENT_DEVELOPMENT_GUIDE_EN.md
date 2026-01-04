# AI Assistant Integration Guide
## Connecting OpenAI Assistants to Your Website

**A Step-by-Step Tutorial for University Students**

---

## Table of Contents

1. [Overview: How AI Assistants Work with Websites](#overview)
2. [Step 1: Create OpenAI API Key](#step-1-create-openai-api-key)
3. [Step 2: Create Your AI Assistant](#step-2-create-your-ai-assistant)
4. [Step 3: Configure Assistant Settings](#step-3-configure-assistant-settings)
5. [Step 4: Understanding the Integration Architecture](#step-4-understanding-the-integration-architecture)
6. [Step 5: Backend Integration Workflow](#step-5-backend-integration-workflow)
7. [Step 6: Frontend Integration Workflow](#step-6-frontend-integration-workflow)

---

## Overview: How AI Assistants Work with Websites {#overview}

### The Big Picture

When a user interacts with an AI assistant on your website, here's what happens:

```
User Types Message
       ↓
Frontend (Website) 
       ↓ HTTP Request
Backend Service (Node.js/Express)
       ↓ OpenAI SDK
OpenAI Assistants API
       ↓ Process & Generate Response
Backend Service
       ↓ HTTP Response
Frontend (Website)
       ↓
Display Response to User
```

### Key Components

1. **Frontend (Website)**: User interface where users type messages
2. **Backend Service**: Middle layer that connects frontend to OpenAI
3. **OpenAI Assistants API**: The AI brain that processes messages
4. **Threads**: Conversation history maintained by OpenAI

### Why We Need a Backend

- **Security**: API keys must stay on the server, never in frontend code
- **Control**: Backend can add business logic, validation, rate limiting
- **Flexibility**: Can integrate with databases, external APIs, etc.

---

## Step 1: Create OpenAI API Key

### 1.1 Get Your API Key

1. Go to https://platform.openai.com
2. Sign up or log in to your account
3. Navigate to **API Keys** (https://platform.openai.com/api-keys)
4. Click **"Create new secret key"**
5. **IMPORTANT**: Copy the key immediately - you won't see it again!
   - Format: `sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 1.2 Store Securely

- Save in backend `.env` file (never commit to Git)
- Add `.env` to `.gitignore`
- Never expose in frontend code

---

## Step 2: Create Your AI Assistant

### 2.1 Navigate to Assistants

1. Go to https://platform.openai.com/assistants
2. Click **"Create"** button

### 2.2 Configure Basic Settings

#### **Name**
- Descriptive name for your reference
- Example: `Company Support Assistant`

#### **Description**
- Brief summary of assistant's purpose
- Example: `Helps users with company information and meeting bookings`

#### **Instructions (System Context)**
This is the most important field - it defines your assistant's behavior.

**Example:**
```
You are a friendly customer support assistant for Intelligent Loops.

Your responsibilities:
- Answer questions about company services
- Help users book meetings
- Provide information about products and pricing

Guidelines:
- Always be professional and helpful
- When booking meetings, check availability first
- Confirm all details before finalizing bookings
```

**Key Points:**
- Be specific about the assistant's role
- Define how it should behave
- Mention when to use functions (if applicable)
- Set the tone (professional, friendly, etc.)

### 2.3 Save and Get Assistant ID

1. Click **"Save"**
2. **Copy the Assistant ID** (starts with `asst_...`)
   - Example: `asst_abc123xyz789`
   - You'll need this for backend configuration

---

## Step 3: Configure Assistant Settings

### 3.1 Model Selection

**Available Models:**
- **GPT-4 Turbo**: Best quality, higher cost
- **GPT-4**: High quality, good balance
- **GPT-3.5 Turbo**: Faster, cheaper, good for simple tasks

**Recommendation**: Start with `gpt-4-turbo-preview` or `gpt-4o` for best results

### 3.2 Temperature

Controls how creative/deterministic the responses are.

- **0.0 - 0.3**: Very focused, consistent (factual information)
- **0.4 - 0.7**: Balanced (Recommended for customer support)
- **0.8 - 1.2**: More creative, varied responses
- **1.3 - 2.0**: Very creative, unpredictable

**Recommended**: `0.7` for most use cases

### 3.3 Top P (Alternative to Temperature)

- Range: 0.0 to 1.0
- Controls diversity of responses
- Use either Temperature OR Top P, not both
- **Recommended**: Leave at default (1.0) or set to `0.7`

### 3.4 Tools (Functions)

This is where you define what functions your assistant can call.

#### **Adding Functions**

1. Click **"Add tool"** → Select **"Function"**
2. Define function in JSON Schema format

**Example Function Definition:**
```json
{
  "type": "function",
  "function": {
    "name": "check_availability",
    "description": "Check if a date and time slot is available for booking",
    "parameters": {
      "type": "object",
      "properties": {
        "date": {
          "type": "string",
          "description": "Date in YYYY-MM-DD format"
        },
        "time": {
          "type": "string",
          "description": "Time in HH:MM format"
        }
      },
      "required": ["date", "time"]
    }
  }
}
```

**Key Components:**
- **name**: Must match your backend function name
- **description**: AI uses this to decide when to call the function
- **parameters**: Define what inputs the function needs
- **required**: List of required parameters

**Common Functions:**
- `check_availability` - Check meeting availability
- `get_current_date_time` - Get current date/time
- `book_meeting` - Book a meeting

### 3.5 File Search (Optional)

- Allows assistant to search through uploaded documents
- Useful for knowledge bases, FAQs, documentation
- Click **"Add tool"** → Select **"File search"** → Upload files

### 3.6 Save Configuration

Review all settings and click **"Save"**

---

## Step 4: Understanding the Integration Architecture

### 4.1 The Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│         FRONTEND (Website)               │
│  - User types message                   │
│  - Sends HTTP POST to backend           │
│  - Displays response                    │
└──────────────┬──────────────────────────┘
               │ HTTP Request
               │ { messages, threadId }
               ▼
┌─────────────────────────────────────────┐
│         BACKEND SERVICE                  │
│  - Receives request from frontend       │
│  - Connects to OpenAI API               │
│  - Handles function calls               │
│  - Returns response to frontend         │
└──────────────┬──────────────────────────┘
               │ OpenAI SDK
               │ openai.beta.threads.*
               ▼
┌─────────────────────────────────────────┐
│      OPENAI ASSISTANTS API              │
│  - Processes messages                   │
│  - Maintains conversation threads       │
│  - Calls functions when needed          │
│  - Generates responses                  │
└─────────────────────────────────────────┘
```

### 4.2 Data Flow

**Step-by-Step Flow:**

1. **User sends message** → Frontend captures input
2. **Frontend → Backend**: HTTP POST with message and threadId
3. **Backend → OpenAI**: Create/retrieve thread, add message, run assistant
4. **OpenAI processes**: May call functions if needed
5. **Backend handles functions**: Executes business logic
6. **Backend → OpenAI**: Submit function results
7. **OpenAI generates response**: Final answer ready
8. **Backend → Frontend**: Return response and threadId
9. **Frontend displays**: Show response to user

### 4.3 Threads Explained

**What is a Thread?**
- A conversation container maintained by OpenAI
- Stores all messages in a conversation
- Each conversation gets a unique `threadId`

**Why Threads Matter:**
- Maintains conversation context
- Allows multi-turn conversations
- Frontend stores `threadId` to continue conversations

**Thread Lifecycle:**
1. First message: Create new thread (threadId = null)
2. OpenAI returns new threadId
3. Subsequent messages: Use existing threadId
4. Conversation continues with full context

---

## Step 5: Backend Integration Workflow

### 5.1 Backend Setup Overview

Your backend needs to:
1. Receive requests from frontend
2. Connect to OpenAI using SDK
3. Manage threads and messages
4. Handle function calls
5. Return responses to frontend

### 5.2 Initialize OpenAI SDK

**Install Package:**
```bash
npm install openai
```

**Initialize Client:**
```javascript
require('dotenv').config();
const OpenAI = require('openai');

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY  // From .env file
});
```

### 5.3 Environment Configuration

Create `.env` file:
```env
OPENAI_API_KEY=sk-proj-your-key-here
```

### 5.4 Core Integration Workflow

Here's the essential workflow your backend needs to implement:

#### **Step 1: Create or Retrieve Thread**

```javascript
let thread_id = threadId; // From frontend request

if (!thread_id) {
  // Create new thread for new conversation
  const thread = await openai.beta.threads.create();
  thread_id = thread.id;
}
```

#### **Step 2: Add User Message**

```javascript
const userMessage = messages[messages.length - 1].content;

await openai.beta.threads.messages.create(thread_id, {
  role: "user",
  content: userMessage
});
```

#### **Step 3: Run Assistant**

```javascript
const run = await openai.beta.threads.runs.create(thread_id, {
  assistant_id: assistantId  // From your .env
});
```

#### **Step 4: Poll for Completion**

```javascript
let runStatus = await openai.beta.threads.runs.retrieve(thread_id, run.id);

while (runStatus.status !== 'completed') {
  // Handle different statuses
  if (runStatus.status === 'requires_action') {
    // Function calls needed - handle them
  }
  
  // Wait and check again
  await new Promise(resolve => setTimeout(resolve, 1000));
  runStatus = await openai.beta.threads.runs.retrieve(thread_id, run.id);
}
```

#### **Step 5: Get Response**

```javascript
const messages_response = await openai.beta.threads.messages.list(thread_id);
const lastMessage = messages_response.data[0];
const responseContent = lastMessage.content[0].text.value;

return { 
  content: responseContent, 
  threadId: thread_id 
};
```

### 5.5 API Endpoint Structure

Your backend needs an endpoint like:

```
POST /api/chat
Body: {
  messages: [{ role: 'user', content: 'Hello' }],
  threadId: null  // or existing threadId
}
Response: {
  content: "Hello! How can I help you?",
  threadId: "thread_abc123"
}
```

### 5.6 Key Backend Responsibilities

1. **Thread Management**: Create new threads, use existing ones
2. **Message Handling**: Add user messages, retrieve assistant responses
3. **Function Calling**: Execute functions when assistant requests them
4. **Error Handling**: Handle API errors gracefully
5. **Security**: Protect API keys, validate inputs

---

## Step 6: Frontend Integration Workflow

### 6.1 Frontend Setup Overview

Your frontend needs to:
1. Capture user input
2. Send HTTP requests to backend
3. Display responses
4. Maintain threadId for conversation continuity

### 6.2 API Configuration

**Create API config:**
```javascript
const API_CONFIG = {
  CHAT_API: 'http://localhost:3001/api/chat'
};
```

### 6.3 Sending Messages

**Basic Flow:**
```javascript
const sendMessage = async (userMessage) => {
  // Prepare messages array
  const messages = [
    { role: 'user', content: userMessage }
  ];
  
  // Send to backend
  const response = await fetch(API_CONFIG.CHAT_API, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
      messages: messages, 
      threadId: threadId  // null for new, or existing threadId
    })
  });
  
  const data = await response.json();
  
  // Display response
  displayMessage(data.content);
  
  // Store threadId for next message
  threadId = data.threadId;
};
```

### 6.4 Maintaining Conversation Context

**Store threadId:**
- First message: `threadId = null`
- Backend returns new `threadId`
- Store in component state or localStorage
- Use same `threadId` for subsequent messages

**Example:**
```javascript
let threadId = null; // Start with null

// First message
const response1 = await sendMessage("Hello");
threadId = response1.threadId; // Store it

// Second message (uses same threadId)
const response2 = await sendMessage("Tell me more", threadId);
// Conversation context is maintained!
```

### 6.5 User Experience Considerations

1. **Loading States**: Show "typing..." indicator while waiting
2. **Error Handling**: Display friendly error messages
3. **Message History**: Show conversation history
4. **Thread Management**: Option to start new conversation


## Key Takeaways

### Integration Essentials

1. **Backend is Required**: Never call OpenAI API directly from frontend
2. **Threads Maintain Context**: Store threadId to continue conversations
3. **Function Calling Enables Actions**: Connect AI to your business logic
4. **Security First**: Keep API keys on backend only

### Workflow Summary

```
Frontend → Backend → OpenAI → Backend → Frontend
   ↓         ↓         ↓         ↓         ↓
User    HTTP    Process   Function  Display
Input   Request Message   Results   Response
```

### Best Practices

- ✅ Store API keys in environment variables
- ✅ Handle errors gracefully
- ✅ Maintain conversation threads
- ✅ Test function calling thoroughly
- ✅ Monitor API usage and costs
- ✅ Provide loading states in UI
- ✅ Validate all inputs

---

## Resources

- **OpenAI Assistants API**: https://platform.openai.com/docs/assistants
- **OpenAI Node.js SDK**: https://github.com/openai/openai-node
- **Function Calling Guide**: https://platform.openai.com/docs/guides/function-calling
- **API Reference**: https://platform.openai.com/docs/api-reference

---
