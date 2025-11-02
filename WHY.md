# Why Another Claude Code Provider?

## The Story Behind @sylphx/ai-sdk-provider-claude-code

### The Existing Landscape

The community already has [`ai-sdk-provider-claude-code`](https://github.com/ben-vargas/ai-sdk-provider-claude-code) by ben-vargas. It's a solid provider that connects Vercel AI SDK to Claude Code CLI. So why did we build another one?

---

## The Fundamental Difference

### What ben-vargas's Provider Does

Ben's provider is a **pure passthrough** - it connects Vercel AI SDK to Claude Code CLI and lets Claude Code handle everything:

```typescript
import { claudeCode } from 'ai-sdk-provider-claude-code';

// This works great for basic text generation
const { text } = await generateText({
  model: claudeCode('sonnet'),
  prompt: 'Hello!'
});

// But when you add Vercel AI SDK tools...
const { text } = await generateText({
  model: claudeCode('sonnet'),
  tools: {
    getWeather: { /* ... */ }  // ❌ This tool is IGNORED
  }
});
```

**The Problem**: Vercel AI SDK tools are completely ignored. Claude Code continues using its own built-in tools (Bash, Read, Write, etc.).

### Why This Happens

Claude Code has its own tool system that works differently from Vercel AI SDK:

1. **Claude Code Tools**: Built-in tools like `Bash`, `Read`, `Write` that execute locally
2. **MCP Servers**: Model Context Protocol servers that Claude Code can connect to
3. **Vercel AI SDK Tools**: JSON Schema-based tools that ben-vargas's provider doesn't translate

Ben's provider passes your Vercel tools to Claude Code, but Claude Code doesn't understand them. It just ignores them and uses its own tools instead.

---

## Our Approach: Vercel-First Philosophy

We built `@sylphx/ai-sdk-provider-claude-code` with a different philosophy:

### 🎯 Vercel AI SDK as the Center

We want the **Vercel AI SDK ecosystem to be the single source of truth** for tools, not Claude Code's system.

```typescript
import { claudeCode } from '@sylphx/ai-sdk-provider-claude-code';

// Your Vercel AI SDK tools WORK! ✅
const { text, toolCalls } = await generateText({
  model: claudeCode('sonnet'),
  tools: {
    getWeather: {
      description: 'Get weather for a city',
      parameters: z.object({ city: z.string() }),
      execute: async ({ city }) => {
        // Your code runs here!
        return await weatherAPI.get(city);
      }
    }
  }
});
```

### How We Achieved This

1. **Disable All Claude Code Tools**: We explicitly disable Claude Code's built-in tools
   ```typescript
   const CLAUDE_CODE_BUILTIN_TOOLS = [
     'Task', 'Bash', 'Glob', 'Grep', 'Read', 'Write', ...
   ];
   queryOptions.disallowedTools = CLAUDE_CODE_BUILTIN_TOOLS;
   ```

2. **Translate Tools to XML**: We convert Vercel AI SDK tool schemas to Claude Code's XML format
   ```typescript
   // Vercel AI SDK tool
   { name: 'getWeather', parameters: { city: string } }

   // Becomes Claude Code XML
   <tool_use>
     <tool_name>getWeather</tool_name>
     <arguments>{"city": "San Francisco"}</arguments>
   </tool_use>
   ```

3. **Parse Tool Calls from Text**: We parse Claude's XML response to extract tool calls
   ```typescript
   const parser = new StreamingXMLParser();
   // Parses <tool_use> blocks from streaming text
   // Emits tool-call events to Vercel AI SDK
   ```

4. **Execute Tools in Vercel Framework**: Tools run in YOUR code, not Claude Code CLI
   ```typescript
   // Your execute function is called by Vercel AI SDK
   execute: async ({ city }) => {
     return await yourAPI.getWeather(city);  // Runs in your environment
   }
   ```

---

## The Technical Trade-off

### Ben-vargas's Provider
- ✅ **Simple**: Pure passthrough, minimal code
- ✅ **Stable**: Relies on Claude Code's mature tool system
- ❌ **Limited**: Only works with MCP servers
- ❌ **Locked-in**: Must use Claude Code's tool ecosystem

### Our Provider (@sylphx)
- ✅ **Universal**: Works with ANY Vercel AI SDK tool
- ✅ **Vercel-First**: Tools execute in your environment
- ✅ **Flexible**: Use the vast Vercel AI SDK tool ecosystem
- ⚠️ **Complex**: Custom XML parser and translation layer
- ⚠️ **Newer**: Less battle-tested than ben-vargas

---

## Why Claude Code Subscription Still Required?

You might ask: "If we're disabling Claude Code's tools, why do we need Claude Code at all?"

### The Answer: Authentication & Model Access

Claude Code CLI provides:

1. **OAuth Authentication**: Access Claude models via your Pro/Max subscription
2. **No API Keys**: Use your existing Claude subscription, no separate API billing
3. **Model Access**: Claude 4.1 Opus, 4.5 Sonnet, 4.5 Haiku

We use Claude Code for **model access**, but not for **tool execution**.

```
┌─────────────────────────────────────────────────────┐
│  Your Application (Vercel AI SDK)                   │
│  ┌───────────────────────────────────┐              │
│  │  Tools Execute Here               │              │
│  │  ✅ getWeather()                  │              │
│  │  ✅ searchDatabase()              │              │
│  │  ✅ sendEmail()                   │              │
│  └───────────────────────────────────┘              │
│           │                                          │
│           │ Tool Results                             │
│           ▼                                          │
│  ┌───────────────────────────────────┐              │
│  │  @sylphx/ai-sdk-provider          │              │
│  │  • Translates tools to XML        │              │
│  │  • Parses tool calls from text    │              │
│  └───────────────────────────────────┘              │
│           │                                          │
│           │ XML + Tool Results                       │
│           ▼                                          │
└───────────┼──────────────────────────────────────────┘
            │
            ▼
    ┌───────────────────────┐
    │  Claude Code CLI       │
    │  • OAuth Auth          │
    │  • Model Access        │
    │  • NO Tool Execution   │
    └───────────────────────┘
```

---

## Our Philosophy

### Vercel AI SDK is the Future

The Vercel AI SDK has:
- 🔧 Rich tool ecosystem
- 🏗️ Standardized interfaces
- 🔄 Framework-agnostic
- 📦 Growing community

We believe tools should live in **your application**, not in the LLM provider's environment.

### Claude Code's Value

Claude Code provides:
- 🔐 Subscription-based access
- 💰 No per-token billing
- 🚀 Latest Claude models

We want to use Claude Code for what it's best at (model access), while keeping tools in Vercel AI SDK where they belong.

---

## Use Cases Comparison

### Use ben-vargas's Provider When:

✅ You want Claude Code's built-in tools (Bash, Read, Write)
✅ You use MCP servers
✅ You want maximum stability
✅ You don't need Vercel AI SDK tools

### Use Our Provider (@sylphx) When:

✅ You want to use Vercel AI SDK tools
✅ You want tools to execute in your application
✅ You want to leverage the Vercel AI SDK ecosystem
✅ You want full control over tool execution
✅ You're building a Vercel AI SDK-first application

---

## Example: The Difference

### With ben-vargas's Provider

```typescript
import { claudeCode } from 'ai-sdk-provider-claude-code';

const { text } = await generateText({
  model: claudeCode('sonnet'),
  prompt: 'Get the weather in San Francisco',
  tools: {
    getWeather: {
      description: 'Get weather',
      parameters: z.object({ city: z.string() }),
      execute: async ({ city }) => {
        return await weatherAPI.get(city);
      }
    }
  }
});

// Result: Claude Code ignores your tool
// It might try to use Bash or other built-in tools instead
// Your getWeather function is NEVER called
```

### With Our Provider (@sylphx)

```typescript
import { claudeCode } from '@sylphx/ai-sdk-provider-claude-code';

const { text, toolCalls } = await generateText({
  model: claudeCode('sonnet'),
  prompt: 'Get the weather in San Francisco',
  tools: {
    getWeather: {
      description: 'Get weather',
      parameters: z.object({ city: z.string() }),
      execute: async ({ city }) => {
        return await weatherAPI.get(city);  // ✅ CALLED!
      }
    }
  }
});

// Result: Your tool is called!
// toolCalls = [{ toolName: 'getWeather', input: { city: 'San Francisco' } }]
// Your execute function runs in your environment
```

---

## Technical Challenges We Solved

### 1. XML Parsing During Streaming

Claude Code streams text that may contain XML tool calls. We built a custom parser that:
- Handles tags split across chunks
- Maintains proper event ordering
- Uses safety margins to prevent parsing errors

### 2. Tool Schema Translation

We translate Vercel's JSON Schema to Claude's expected XML format:
```typescript
// Input: Vercel AI SDK tool
{
  name: 'getWeather',
  description: 'Get weather for a location',
  parameters: {
    type: 'object',
    properties: { city: { type: 'string' } }
  }
}

// Output: Claude Code system prompt
Available tools:
- getWeather: Get weather for a location
  Parameters: { city: string }
```

### 3. Result Formatting

We format tool results back into XML for Claude:
```typescript
<tool_result>
  <tool_call_id>call_1</tool_call_id>
  <result>
    {"temperature": 72, "condition": "sunny"}
  </result>
</tool_result>
```

---

## Conclusion

We didn't build another Claude Code provider because ben-vargas's provider is bad. We built it because we have a **different vision**:

- **Ben's approach**: Use Claude Code's ecosystem → Stable, simple, limited
- **Our approach**: Use Vercel AI SDK's ecosystem → Flexible, powerful, complex

Both are valid. Choose based on your needs:

- Want Claude Code tools? → Use ben-vargas
- Want Vercel AI SDK tools? → Use @sylphx

---

## Credits

We deeply respect ben-vargas's work. His provider is solid and serves its purpose well. We simply have different goals:

- **ben-vargas**: Connect Vercel AI SDK to Claude Code (pure passthrough)
- **@sylphx**: Make Claude Code work like a standard Vercel AI SDK provider (full translation)

Both providers are open source. Both serve the community. Choose the one that fits your use case! 🚀

---

**Made with respect for ben-vargas's excellent work by Sylph X Ltd**
