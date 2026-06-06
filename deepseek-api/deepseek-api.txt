================================================================================
                         DEEPSEEK API DETAILED GUIDE
================================================================================

TABLE OF CONTENTS
───────────────────────────────────────────────────────────────────────────────
1. What is DeepSeek?
2. Available Models
3. API Authentication
4. Base URL Endpoints
5. Chat Completions Endpoint
6. Streaming vs Non-Streaming
7. Token Limits
8. Pricing
9. Rate Limits
10. Integration with OpenRouter
11. Using with Cline as a Provider
12. Example Requests

================================================================================
1. WHAT IS DEEPSEEK?
================================================================================

DeepSeek is a Chinese AI research company focused on large language models.
They have gained significant attention for producing highly capable models with
efficient architectures and competitive pricing.

Key highlights:
  • Founded by High-Flyer (quantitative hedge fund).
  • Known for Mixture-of-Experts (MoE) architecture innovations.
  • Models rival GPT-4 and Claude in many benchmarks.
  • Significantly cheaper than OpenAI and Anthropic.
  • Open-source weights available for many models.
  • Strong focus on reasoning and coding capabilities.

DeepSeek provides both:
  1. A web chat interface (chat.deepseek.com) for direct interaction.
  2. A REST API for programmatic access (api.deepseek.com).

This guide focuses on the REST API for developers.

================================================================================
2. AVAILABLE MODELS
================================================================================

2.1 DeepSeek-V2 / DeepSeek-V2.5
───────────────────────────────────────────────────────────────────────────────
  • Model ID: deepseek-chat (or deepseek-v2)
  • Architecture: Mixture-of-Experts (MoE)
  • Total parameters: ~236B, activated per token: ~21B
  • Context window: 128K tokens
  • Strengths: General purpose, instruction following, long context
  • Status: Stable, recommended for most use cases

2.2 DeepSeek-Coder-V2
───────────────────────────────────────────────────────────────────────────────
  • Model ID: deepseek-coder
  • Architecture: MoE-based, specialized for code
  • Total parameters: ~236B
  • Context window: 128K tokens
  • Strengths: Code generation, debugging, code completion, multi-file
    refactoring. Supports all major programming languages.
  • Training: Pretrained on 2T tokens of code and natural language.
  • Benchmarks: Competitive with GPT-4 Turbo on coding tasks (HumanEval,
    MBPP, SWE-Bench).

2.3 DeepSeek-R1
───────────────────────────────────────────────────────────────────────────────
  • Model ID: deepseek-reasoner (or deepseek-r1)
  • Architecture: MoE with chain-of-thought reasoning
  • Parameters: ~685B total (MoE), ~37B activated per token

================================================================================
3. API AUTHENTICATION
================================================================================

DeepSeek API uses API key authentication.

3.1 Getting an API Key
───────────────────────────────────────────────────────────────────────────────
  1. Go to https://platform.deepseek.com
  2. Sign up / log in.
  3. Navigate to API Keys section.
  4. Click "Create API Key".
  5. Copy the key immediately — it is only shown once.
  6. Store it securely (environment variable, secrets manager, etc.).

3.2 Using the API Key
───────────────────────────────────────────────────────────────────────────────
The API key is sent as a Bearer token in the Authorization header:

  Authorization: Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

3.3 Best Practices
───────────────────────────────────────────────────────────────────────────────
  • Never hardcode API keys in source code.
  • Use environment variables: DEEPSEEK_API_KEY
  • On Windows PowerShell: $env:DEEPSEEK_API_KEY = "sk-..."
  • On Linux/macOS: export DEEPSEEK_API_KEY="sk-..."
  • Rotate keys periodically via the platform dashboard.
  • Use different keys for development and production.

================================================================================
4. BASE URL ENDPOINTS
================================================================================

4.1 Official DeepSeek API
───────────────────────────────────────────────────────────────────────────────
  Base URL: https://api.deepseek.com
  API Version: v1 (OpenAI-compatible)

4.2 Key Endpoints
───────────────────────────────────────────────────────────────────────────────
  Chat Completions:
    POST https://api.deepseek.com/v1/chat/completions

  List Models:
    GET https://api.deepseek.com/v1/models

  Check Balance:
    POST https://api.deepseek.com/v1/dashboard/balance
    (May vary; check platform documentation.)

4.3 OpenAI Compatibility
───────────────────────────────────────────────────────────────────────────────
DeepSeek's API is designed to be compatible with the OpenAI API format. This
means:
  • You can use OpenAI client libraries with DeepSeek by changing the base URL.
  • Request/response formats are nearly identical.
  • Most OpenAI SDKs work with minimal modification.

  Example with OpenAI Python SDK:
    from openai import OpenAI
    client = OpenAI(
        api_key="sk-...",
        base_url="https://api.deepseek.com/v1"
    )

================================================================================
5. CHAT COMPLETIONS ENDPOINT
───────────────────────────────────────────────────────────────────────────────

5.1 Request Format
───────────────────────────────────────────────────────────────────────────────
  POST https://api.deepseek.com/v1/chat/completions

  Headers:
    Content-Type: application/json
    Authorization: Bearer sk-...

  Body (JSON):
    {
      "model": "deepseek-chat",
      "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"}
      ],
      "temperature": 0.7,
      "max_tokens": 1024,
      "top_p": 1.0,
      "frequency_penalty": 0.0,
      "presence_penalty": 0.0,
      "stream": false
    }

5.2 Parameters
───────────────────────────────────────────────────────────────────────────────
  model (string, required):
    Model ID. Options: "deepseek-chat", "deepseek-coder", "deepseek-reasoner"

  messages (array, required):
    Array of message objects with "role" and "content".
    Roles: "system", "user", "assistant"

  temperature (float, optional, default: 1.0):
    Controls randomness. Range: 0.0 to 2.0.
    Lower = more deterministic, higher = more creative.

  top_p (float, optional, default: 1.0):

================================================================================
6. STREAMING vs NON-STREAMING
================================================================================

6.1 Non-Streaming
───────────────────────────────────────────────────────────────────────────────
  • Set stream: false (or omit the parameter).
  • The API returns the complete response in a single HTTP response.
  • Simpler to implement; works with any HTTP client.
  • Better for: Short responses, non-interactive use, logging/auditing.
  • Trade-off: Higher latency perception (wait for full response).

6.2 Streaming (SSE)
───────────────────────────────────────────────────────────────────────────────
  • Set stream: true.
  • The API sends data as Server-Sent Events (text/event-stream).
  • Each chunk contains a delta of the response.
  • Client reassembles chunks into the full response.

  Stream chunk format:
    data: {"choices":[{"delta":{"content":"Paris"},"index":0}]}
    data: {"choices":[{"delta":{"content":" is"},"index":0}]}
    data: {"choices":[{"delta":{"content":" the"},"index":0}]}
    ...
    data: [DONE]

  • Better for: Real-time UI, chatbots, progressive display.
  • Trade-off: More complex client-side handling.

6.3 DeepSeek-R1 Streaming Notes
───────────────────────────────────────────────────────────────────────────────
When streaming deepseek-reasoner, you may receive reasoning content as a
separate field in the delta:

  data: {"choices":[{"delta":{"reasoning":"Let me think..."}}]}
  data: {"choices":[{"delta":{"reasoning":"First, calculate X..."}}]}
  data: {"choices":[{"delta":{"content":"The answer is"}}]}

Your streaming client should handle both "content" and "reasoning" deltas.

================================================================================
7. TOKEN LIMITS
───────────────────────────────────────────────────────────────────────────────

7.1 Context Window
───────────────────────────────────────────────────────────────────────────────
All DeepSeek models support a context window of 128K tokens. This includes
both input (prompt) and output (completion) tokens.

  128K tokens ≈ 96,000 English words ≈ 200+ pages of text.

7.2 Max Output Tokens
───────────────────────────────────────────────────────────────────────────────
The maximum output length (max_tokens) varies:
  • deepseek-chat:     Up to 8,192 tokens
  • deepseek-coder:    Up to 8,192 tokens
  • deepseek-reasoner: Up to 8,192 tokens

Note: For deepseek-reasoner, the reasoning tokens count toward output,
so the visible answer may be shorter than max_tokens.

7.3 Token Counting
───────────────────────────────────────────────────────────────────────────────
Use the response's usage object to track tokens:

  "usage": {
    "prompt_tokens": 50,
    "completion_tokens": 100,
    "total_tokens": 150

================================================================================
9. RATE LIMITS
───────────────────────────────────────────────────────────────────────────────

9.1 Default Limits
───────────────────────────────────────────────────────────────────────────────
DeepSeek API rate limits depend on your account tier and balance.

Typical limits (may vary):
  • Free tier:       10 RPM (requests per minute), 10K TPM (tokens per minute)
  • Paid tier:       60 RPM, 100K TPM (increases with usage history)
  • Higher tiers:    500+ RPM, 1M+ TPM (contact sales for enterprise)

9.2 Rate Limit Headers
───────────────────────────────────────────────────────────────────────────────
The API returns rate limit info in response headers:
  X-RateLimit-Limit:     60
  X-RateLimit-Remaining: 45
  X-RateLimit-Reset:     1700000000

9.3 Handling Rate Limits
───────────────────────────────────────────────────────────────────────────────
When rate limited, the API returns HTTP 429 (Too Many Requests).

  Response:
    {
      "error": {
        "message": "Rate limit exceeded. Please retry after X seconds.",
        "type": "rate_limit_error",
        "param": null,
        "code": "rate_limit_exceeded"
      }
    }

Best practices:
  • Implement exponential backoff with jitter.
  • Queue requests and process at a controlled rate.
  • Monitor X-RateLimit-Remaining and preemptively slow down.
  • Use request batching where possible.
  • Consider upgrading your tier if consistently hitting limits.

  Python example with exponential backoff:
    import time
    import random

    def api_call_with_retry(client, **kwargs):
        max_retries = 5
        for attempt in range(max_retries):
            try:
                return client.chat.completions.create(**kwargs)
            except Exception as e:
                if attempt == max_retries - 1:
                    raise
                wait = (2 ** attempt) + random.random()
                time.sleep(wait)

================================================================================
10. INTEGRATION WITH OPENROUTER
───────────────────────────────────────────────────────────────────────────────

OpenRouter provides a unified API to access multiple LLM providers, including
DeepSeek models, without managing separate API keys.

10.1 Benefits of Using OpenRouter for DeepSeek
───────────────────────────────────────────────────────────────────────────────
  • Single API key for all providers.
  • Automatic fallback if one provider is down.
  • Unified billing across providers.
  • Access to DeepSeek models without a DeepSeek account.

================================================================================
12. EXAMPLE REQUESTS
───────────────────────────────────────────────────────────────────────────────

12.1 cURL — Basic Chat Completion
───────────────────────────────────────────────────────────────────────────────
  curl https://api.deepseek.com/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $DEEPSEEK_API_KEY" \
    -d '{
      "model": "deepseek-chat",
      "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Write a Python function to reverse a string."}
      ],
      "temperature": 0.3,
      "max_tokens": 500
    }'

12.2 cURL — Streaming
───────────────────────────────────────────────────────────────────────────────
  curl https://api.deepseek.com/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $DEEPSEEK_API_KEY" \
    -d '{
      "model": "deepseek-chat",
      "messages": [{"role": "user", "content": "Count from 1 to 5."}],
      "stream": true
    }'

12.3 Python — Basic Request (using openai library)
───────────────────────────────────────────────────────────────────────────────
  from openai import OpenAI

  client = OpenAI(
      api_key="sk-your-deepseek-api-key",
      base_url="https://api.deepseek.com/v1"
  )

  response = client.chat.completions.create(
      model="deepseek-chat",
      messages=[
          {"role": "system", "content": "You are a coding expert."},
          {"role": "user", "content": "Explain async/await in Python."}
      ],
      temperature=0.5,
      max_tokens=1000
  )

  print(response.choices[0].message.content)

12.4 Python — Streaming
───────────────────────────────────────────────────────────────────────────────
  from openai import OpenAI

  client = OpenAI(
      api_key="sk-your-deepseek-api-key",
      base_url="https://api.deepseek.com/v1"
  )

  stream = client.chat.completions.create(
      model="deepseek-chat",
      messages=[{"role": "user", "content": "Write a haiku about coding."}],
      stream=True
  )

  for chunk in stream:
      if chunk.choices[0].delta.content:
          print(chunk.choices[0].delta.content, end="", flush=True)

12.5 Python — DeepSeek-R1 with Reasoning
───────────────────────────────────────────────────────────────────────────────
  from openai import OpenAI

  client = OpenAI(
      api_key="sk-your-deepseek-api-key",
      base_url="https://api.deepseek.com/v1"
  )

  response = client.chat.completions.create(
      model="deepseek-reasoner",
      messages=[
          {"role": "user", "content": "If a train leaves station A at 60 mph\nand another leaves station B at 80 mph, 200 miles apart,\nwhen do they meet?"}
      ],
      max_tokens=2000
  )

  msg = response.choices[0].message
  print("Reasoning:", getattr(msg, 'reasoning', 'N/A'))
  print("Answer:", msg.content)

12.6 Python — Using Requests Library (No OpenAI SDK)
───────────────────────────────────────────────────────────────────────────────
  import requests
  import json

  url = "https://api.deepseek.com/v1/chat/completions"
  headers = {
      "Content-Type": "application/json",
      "Authorization": "Bearer sk-your-deepseek-api-key"
  }
  data = {
      "model": "deepseek-chat",
      "messages": [
          {"role": "user", "content": "What is the meaning of life?"}
      ],
      "temperature": 0.7,
      "max_tokens": 200
  }

  response = requests.post(url, headers=headers, json=data)
  print(response.json()["choices"][0]["message"]["content"])

12.7 Node.js — Using OpenAI SDK
───────────────────────────────────────────────────────────────────────────────
  import OpenAI from 'openai';

  const client = new OpenAI({
      apiKey: 'sk-your-deepseek-api-key',
      baseURL: 'https://api.deepseek.com/v1'
  });

  const response = await client.chat.completions.create({
      model: 'deepseek-chat',
      messages: [
          { role: 'user', content: 'What is the capital of Japan?' }
      ]
  });

  console.log(response.choices[0].message.content);

12.8 Error Handling Template
───────────────────────────────────────────────────────────────────────────────
  import requests

  def deepseek_chat(messages, model="deepseek-chat", **kwargs):
      url = "https://api.deepseek.com/v1/chat/completions"
      headers = {
          "Content-Type": "application/json",
          "Authorization": f"Bearer {DEEPSEEK_API_KEY}"
      }
      data = {
          "model": model,
          "messages": messages,
          **kwargs
      }

      try:
          resp = requests.post(url, headers=headers, json=data, timeout=60)
          resp.raise_for_status()
          return resp.json()

      except requests.exceptions.HTTPError as e:
          error_body = resp.json()
          if resp.status_code == 429:
              print("Rate limited. Retrying...")
              # Implement retry logic here
          elif resp.status_code == 401:
              print("Invalid API key.")
          elif resp.status_code == 400:
              print(f"Bad request: {error_body}")
          else:
              print(f"HTTP {resp.status_code}: {error_body}")
          raise

      except requests.exceptions.Timeout:
          print("Request timed out.")
          raise

      except requests.exceptions.ConnectionError:
          print("Connection error. Check network/endpoint.")
          raise

================================================================================
END OF DEEPSEEK API DETAILED GUIDE
================================================================================

  • Additional features: prompt caching, model routing.

10.2 Setup
───────────────────────────────────────────────────────────────────────────────
  Endpoint: https://openrouter.ai/api/v1/chat/completions
  API Key:  Get from https://openrouter.ai/keys

  Headers:
    Authorization: Bearer <openrouter-key>
    Content-Type: application/json

  Body: Same format as DeepSeek/OpenAI API.
    Set "model" to the OpenRouter model ID:
      • "deepseek/deepseek-chat"
      • "deepseek/deepseek-coder"
      • "deepseek/deepseek-reasoner"

10.3 Provider Routing (Optional)
───────────────────────────────────────────────────────────────────────────────
OpenRouter can route through specific providers:

  {
    "model": "deepseek/deepseek-chat",
    "provider": {
      "order": ["DeepSeek"],
      "allow_fallbacks": false
    }
  }

10.4 Pricing Transparency
───────────────────────────────────────────────────────────────────────────────
OpenRouter shows the exact per-token cost on each response. Check
https://openrouter.ai/deepseek for current pricing.

================================================================================
11. USING WITH CLINE AS A PROVIDER
───────────────────────────────────────────────────────────────────────────────

You can use DeepSeek models as the AI provider for Cline.

11.1 Configuration Steps
───────────────────────────────────────────────────────────────────────────────
  1. Open Cline settings.
  2. Under API Provider, select "OpenAI Compatible".
  3. Set the following:
     Base URL: https://api.deepseek.com/v1
     API Key:  Your DeepSeek API key
     Model:    deepseek-chat (or deepseek-coder, deepseek-reasoner)

  4. Click "Save" or "Confirm".

11.2 Recommended Model for Cline
───────────────────────────────────────────────────────────────────────────────
  • deepseek-chat: Good for general coding assistance.
  • deepseek-coder: Best for code-specific tasks (recommended).
  • deepseek-reasoner: Overkill for simple tasks but excellent for complex
    debugging and architecture problems.

11.3 Via OpenRouter
───────────────────────────────────────────────────────────────────────────────
  Provider: "OpenAI Compatible"
  Base URL: https://openrouter.ai/api/v1
  API Key:  Your OpenRouter API key
  Model:    deepseek/deepseek-coder (or deepseek/deepseek-chat)

11.4 Limitations with Cline
───────────────────────────────────────────────────────────────────────────────
  • DeepSeek models may not follow instructions as precisely as Claude.
  • Tool calling quality can vary — test with your workflow.
  • DeepSeek-R1 may be too slow for interactive use in Cline due to
    extended reasoning time.
  • Some Cline features (like checkpoints) work independently of the
    provider model.

  }

You can estimate token count locally:
  • English: ~1 token per 0.75 words
  • Code: ~1 token per 1-2 characters
  • Chinese characters: ~1 token per character

================================================================================
8. PRICING
───────────────────────────────────────────────────────────────────────────────

Pricing is per 1M tokens (as of latest rates — check platform.deepseek.com for
current pricing).

8.1 Approximate Rates
───────────────────────────────────────────────────────────────────────────────
  deepseek-chat:
    Input:  ~$0.14 / 1M tokens
    Output: ~$0.28 / 1M tokens

  deepseek-coder:
    Input:  ~$0.14 / 1M tokens
    Output: ~$0.28 / 1M tokens

  deepseek-reasoner:
    Input:  ~$0.55 / 1M tokens
    Output: ~$2.19 / 1M tokens (includes reasoning tokens)
    Reasoning tokens: ~$0.55 / 1M tokens (billed separately)

8.2 Cost Comparison
───────────────────────────────────────────────────────────────────────────────
Relative to other providers (approximate):
  DeepSeek is roughly:
    • 10-20x cheaper than GPT-4
    • 5-10x cheaper than Claude 3 Opus
    • 2-5x cheaper than Gemini Ultra
    • Comparable to GPT-4o-mini pricing but with higher capability

8.3 Billing
───────────────────────────────────────────────────────────────────────────────
  • Prepaid: Top up your account balance.
  • Minimum top-up: Typically $10 or equivalent.
  • View usage: Platform dashboard → Usage.
  • Alerts: Set budget alerts in the dashboard.

    Nucleus sampling. Alternative to temperature.

  max_tokens (integer, optional, default: 4096):
    Maximum number of tokens to generate.

  stream (boolean, optional, default: false):
    If true, responses are sent as SSE (Server-Sent Events).

  stop (string or array, optional):
    Sequences where generation stops.

  frequency_penalty (float, optional, default: 0.0):
    Penalizes repeated tokens. Range: -2.0 to 2.0.

  presence_penalty (float, optional, default: 0.0):
    Penalizes topic repetition. Range: -2.0 to 2.0.

5.3 Response Format (Non-Streaming)
───────────────────────────────────────────────────────────────────────────────
  {
    "id": "chatcmpl-xxx",
    "object": "chat.completion",
    "created": 1700000000,
    "model": "deepseek-chat",
    "choices": [
      {
        "index": 0,
        "message": {
          "role": "assistant",
          "content": "The capital of France is Paris."
        },
        "finish_reason": "stop"
      }
    ],
    "usage": {
      "prompt_tokens": 30,
      "completion_tokens": 10,
      "total_tokens": 40
    }
  }

5.4 DeepSeek-R1 Special Response
───────────────────────────────────────────────────────────────────────────────
When using deepseek-reasoner, the response includes a "reasoning" field
containing the model's chain-of-thought:

  {
    "choices": [{
      "message": {
        "role": "assistant",
        "content": "The answer is 42.",
        "reasoning": "First, I need to calculate... *extended reasoning* ... Therefore the result is 42."
      }
    }]
  }

  • Context window: 128K tokens
  • Strengths: Mathematical reasoning, complex problem solving, scientific
    analysis. Uses extended chain-of-thought before answering.
  • Key feature: "Think step by step" built into the architecture — the
    model generates internal reasoning tokens before the final answer.
  • Note: Reasoning tokens are billed separately from output tokens.

2.4 Model Selection Guide
───────────────────────────────────────────────────────────────────────────────
  Task                    Recommended Model
  ─────────────────────────────────────────────────────────────────
  General chat/QA         DeepSeek-V2
  Code generation         DeepSeek-Coder-V2
  Bug fixing              DeepSeek-Coder-V2
  Math/reasoning          DeepSeek-R1
  Long document analysis  DeepSeek-V2 (128K context)
  Multi-file refactoring  DeepSeek-Coder-V2
  Scientific research     DeepSeek-R1

