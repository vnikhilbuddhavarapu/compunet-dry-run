# Cloudflare AI Chat App

A chat application built with Cloudflare Workers AI and AI Gateway using OpenAI's GPT-OSS-120b model. Features markdown rendering, advanced reasoning capabilities, and optional content filtering through AI Gateway guardrails.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/acme-studios/chatbot-with-gateway)

## What This Does

This is a chatbot that uses Cloudflare Workers AI with OpenAI's GPT-OSS-120b model to generate responses. GPT-OSS-120b is an open-weight model designed for powerful reasoning, agentic tasks, and production use cases with a 128,000 token context window.

By default, requests go directly to the AI model. You can optionally route requests through AI Gateway to add content filtering, caching, and rate limiting.

The UI supports both light and dark themes, renders markdown responses with syntax highlighting, and handles errors gracefully.

### Model Features
- **Model**: @cf/openai/gpt-oss-120b
- **Context Window**: 128,000 tokens
- **Reasoning Capabilities**: Configurable reasoning effort (low, medium, high)

## Quick Start

```bash
npm install
npm run dev
```

Visit http://localhost:8787 to test locally.

To deploy:
```bash
npm run deploy
```

## Configuring the AI Model

### Current Model: GPT-OSS-120b

The chatbot is configured to use OpenAI's GPT-OSS-120b model. You can customize its behavior in `src/index.ts`:

```typescript
// Model ID
const MODEL_ID = "@cf/openai/gpt-oss-120b";

// Reasoning configuration
const REASONING_EFFORT = "medium"; // Options: "low", "medium", "high"
const REASONING_SUMMARY = "auto";  // Options: "auto", "concise", "detailed"
```

### Reasoning Configuration

**REASONING_EFFORT**: Controls how much computational effort the model spends on reasoning
- `low`: Faster responses, fewer tokens, less thorough reasoning
- `medium`: Balanced performance and reasoning quality (default)
- `high`: Most thorough reasoning, more tokens, slower responses

**REASONING_SUMMARY**: Controls the detail level of reasoning summaries
- `auto`: Model decides the appropriate level
- `concise`: Brief reasoning summaries
- `detailed`: Comprehensive reasoning explanations

### Switching to a Different Model

To use a different Workers AI model, update the `MODEL_ID` constant and adjust the API options accordingly. Note that GPT-OSS-120b uses the Responses API format (with `input` and `instructions` parameters) which differs from the chat/messages API used by some other models.

Find available models at https://developers.cloudflare.com/workers-ai/models/

## Setting Up AI Gateway (Optional)

**By default, the chatbot works without AI Gateway** - all requests go directly to the GPT-OSS-120b model. AI Gateway is completely optional and adds content filtering, caching, and analytics if you need them.

### To Enable AI Gateway:

#### 1. Create an AI Gateway

1. Go to your Cloudflare dashboard
2. Navigate to AI > AI Gateway
3. Click "Create Gateway"
4. Give it a name (e.g., "chatbot-gateway")
5. Save the gateway

#### 2. Configure Guardrails (Optional)

In your gateway settings:

1. Go to the "Guardrails" tab
2. Enable the content filters you want:
   - Hate speech
   - Violence
   - Self-harm
   - Sexual content
   - etc.
3. Save your settings

#### 3. Update Your Code

Open `src/index.ts` and set your gateway ID (currently line 20):

```typescript
const AI_GATEWAY_ID = "chatbot-gateway"; // Your gateway name
```

#### 4. Redeploy

```bash
npm run deploy
```

Now all requests will go through your AI Gateway. Blocked prompts or responses will show detailed error messages in the UI.

### To Disable AI Gateway:

Simply set `AI_GATEWAY_ID = ""` in `src/index.ts` and redeploy. All requests will go directly to the model.

## How It Works

- User sends a message
- If AI Gateway is configured, the request goes through the gateway first
- Gateway checks content against guardrails
- If approved, request goes to Workers AI model
- Response is returned to the user
- If blocked, user sees a detailed error message

Without AI Gateway configured, requests go directly to the model.

## Project Structure

```
src/index.ts       - Backend API and AI logic (GPT-OSS-120b integration)
src/types.ts       - TypeScript definitions
public/index.html  - UI and styling
public/chat.js     - Frontend logic
wrangler.jsonc     - Worker configuration
```

## Technical Details

### GPT-OSS-120b Responses API

This implementation uses the Responses API format required by GPT-OSS-120b:
- **instructions**: System-level guidance (replaces system role messages)
- **input**: Conversation history array (user/assistant messages)
- **reasoning**: Configuration object for reasoning behavior

The code automatically handles:
- Conversation history sanitization
- Blocked content filtering
- Context window management (last 16 messages)
- Response parsing and error handling

## Resources

- [GPT-OSS-120b Model Documentation](https://developers.cloudflare.com/workers-ai/models/gpt-oss-120b/)
- [Workers AI Models](https://developers.cloudflare.com/workers-ai/models/)
- [AI Gateway Documentation](https://developers.cloudflare.com/ai-gateway/)
- [Workers Documentation](https://developers.cloudflare.com/workers/)
- [OpenAI Responses API](https://platform.openai.com/docs/api-reference/responses)
