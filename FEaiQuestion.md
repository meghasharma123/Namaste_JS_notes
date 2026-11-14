## 1. You're building an enterprise AI chat application using React and TypeScript.

The application has:
- A chat input.
- A conversation history.
- AI-generated responses.
- A loading indicator while the AI is processing.
- The ability to start multiple conversations.
How would you structure your React components and manage the application state?

=> I would follow a component-based architecture and divide the application into reusable components such as Sidebar, ConversationList, ChatWindow, MessageList, MessageBubble, and ChatInput.

For state management, I would use local React state for UI-specific data and Zustand or Redux Toolkit for shared data such as conversations, messages, and the active conversation.

For API integration, whenever a user sends a message, I would optimistically update the UI, show a loading state, and send the request to the backend. Once the AI response arrives, I would update the conversation. For a better user experience, I would support streaming responses using SSE or WebSockets and handle errors and connection interruptions.

```
AIChatApp
 ├── Sidebar
 │    ├── ConversationList
 │    └── NewChatButton
 │
 └── ChatWindow
      ├── MessageList
      │    └── MessageBubble
      ├── LoadingIndicator
      └── ChatInput
```

## 2. Suppose the user clicks the New Chat button. Which state would you update to make that new conversation active?

=> When the user clicks New Chat, I would create a new conversation, add it to the global state, and update the activeConversationId. I would also reset local UI states such as the input field, loading indicator, and errors so the user starts with a clean conversation.

## 3. How would you define a TypeScript interface for this message, and how would you use that interface in a React component's props?

```
{
  id: "123",
  role: "assistant",
  content: "Here is your answer",
  timestamp: 1730000000
}
```

=> I would define a reusable TypeScript interface for the chat message structure, specifying properties such as ID, role, content and timestamp. I would export this interface and reuse it across components to maintain type safety and consistency. I would also use union types for restricted values like message roles.

```
export interface ChatMessage {
  id: string;
  role: "user" | "assistant" | "system";
  content: string;
  timestamp: number;
}

interface MessageProps {
  message: ChatMessage;
}

const MessageBubble: React.FC<MessageProps> = ({ message }) => {
  return <div>{message.content}</div>;
};
```

## 4. How would you implement this in React? What technology or communication mechanism would you use?

=> I would use Server-Sent Events (SSE) for streaming AI responses. The client sends a prompt to the backend, and the server streams response chunks as they are generated. In React, I would append each incoming chunk to the existing message state so users can see the response progressively instead of waiting for the complete response. I would also handle connection errors, cancellation and stream completion.

## 5. If you update the React state for every single chunk, it could cause frequent re-renders.How would you optimize the rendering performance in this situation?

=> I would avoid triggering a React state update for every incoming token. I can use useRef to accumulate incoming chunks without causing re-renders and periodically synchronize the accumulated response with useState. This reduces unnecessary rendering while keeping the UI responsive. I would also consider memoizing message components where appropriate.

## 6. Imagine a user asks your enterprise AI application, "What is our company's annual leave policy?" The AI backend uses RAG to retrieve information from company documents and generate an answer.

```
{
  "answer": "Employees receive 20 annual leave days.",
  "sources": [
    {
      "title": "HR Policy 2026",
      "page": 12,
      "content": "Employees are entitled to 20 annual leave days."
    }
  ]
}
```

How would you display this response in React so that users can verify where the AI got its answer from?

=> I would display the AI-generated answer along with clickable citations containing the document title and page number. When users click a citation, I would open a source preview showing the relevant document excerpt. This improves transparency, trust and explainability.

## 9. How would you design the React frontend to show which action the AI agent is performing and its current execution status?

Suppose an enterprise AI assistant can perform actions such as:
- Create a purchase order.
- Send an email.
- Fetch customer information.
The AI decides which tool to call based on the user's prompt.

=> I would create a structured data model for agent actions containing the tool name, execution status and result. The backend would communicate progress updates through SSE or WebSockets. I would update the React state whenever an action status changes and dynamically render progress indicators such as pending, running, completed and failed. This provides transparency into the agent's workflow.

```
interface AgentAction {
  id: string;
  tool: string;
  status: "pending" | "running" | "completed" | "failed";
  message?: string;
}
```

## 10. Suppose the AI agent wants to send an email to a customer or create a purchase order. The company requires the user to approve the action before execution. How would you implement this approval workflow in React, and how would you ensure that the AI cannot execute the action without user approval?

=> I would implement a human-in-the-loop workflow where the AI agent first proposes an action, and the frontend displays a confirmation dialog with all relevant details. The user must explicitly approve or reject the action. I would maintain the approval status in React state, but the backend would independently validate the approval and user permissions before executing any sensitive operation. This ensures security and prevents unauthorized AI actions.

## 11. Imagine your enterprise AI chat application displays AI-generated Markdown and HTML responses. An AI response could potentially contain malicious HTML or JavaScript. How would you prevent XSS (Cross-Site Scripting) attacks while rendering AI-generated content in React?

=> Since AI-generated content is untrusted, I would sanitize HTML using `DOMPurify` before rendering it. I would avoid `dangerouslySetInnerHTML` wherever possible and use a safe Markdown renderer with raw HTML disabled. Additionally, I would implement Content Security Policy headers as another layer of protection against XSS.

## 12. How would you optimize the React application to handle thousands of chat messages efficiently?

=> I would optimize large conversations using `pagination` and `virtualization`. I would initially render only a limited number of messages and fetch older messages when the user scrolls upward. I would use a virtualization library to keep only visible message elements in the DOM. Additionally, I would use caching to avoid unnecessary API requests and React.memo where appropriate to prevent unnecessary re-renders.

## 13. You're building an enterprise AI application using React and TypeScript. Before releasing it to production, what important things would you consider regarding?

=> Before production, I would implement unit, integration and E2E testing using tools like Jest, React Testing Library and Playwright. For error handling, I would use Error Boundaries, API error handling, retry mechanisms and appropriate fallback UI. I would monitor application performance using Lighthouse, React Profiler and Core Web Vitals. For accessibility, I would follow WCAG guidelines, ensuring keyboard navigation, screen-reader compatibility, semantic HTML and sufficient color contrast.

## 14. Code output

```
const [count, setCount] = useState(0);

<button onClick={() => setCount(() => count + 1)}>
  {count}
</button>
```

In React, we use functional updates when the new state depends on the previous state.

*a. Direct update*

`setCount(count + 1);` 
- Uses count from the current render.
- Can cause stale state issues when multiple updates are queued.

*b. Functional update*

`setCount(prev => prev + 1);`
- Receives the latest queued state.
- Safely handles multiple state updates.

```
function handleClick() {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
}
```

