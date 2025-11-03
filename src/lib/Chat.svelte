<script>
  import Anthropic from '@anthropic-ai/sdk';

  let messages = [];
  let inputMessage = '';
  let isLoading = false;
  let messagesContainer;

  // MODEL SELECTION: User can choose between different Claude models
  // Each model has different capabilities, speeds, and costs
  let selectedModel = 'claude-3-5-sonnet-20241022'; // Default to latest Sonnet

  // AVAILABLE MODELS: Claude model variants from Anthropic
  // All models support streaming and extended thinking
  const models = [
    {
      id: 'claude-3-5-haiku-20241022',
      name: 'Claude 3.5 Haiku',
      description: 'Fast & economical - Best for quick conversations'
    },
    {
      id: 'claude-3-5-sonnet-20241022',
      name: 'Claude 3.5 Sonnet',
      description: 'Balanced - Great for most tasks with good speed'
    },
    {
      id: 'claude-opus-4-20250514',
      name: 'Claude Opus 4',
      description: 'Most capable - Best for complex reasoning and analysis'
    }
  ];

  // ANTHROPIC CLIENT: Initialize with API key from environment variable
  // SECURITY: Never hardcode API keys - always use .env file
  // Create a .env file with: VITE_ANTHROPIC_API_KEY=your_key_here
  const anthropic = new Anthropic({
    apiKey: import.meta.env.VITE_ANTHROPIC_API_KEY,
    dangerouslyAllowBrowser: true // Note: In production, use a backend server
  });

  async function sendMessage() {
    // UI GUARD: Prevent sending if input is empty or already waiting for response
    if (!inputMessage.trim() || isLoading) return;

    // CAPTURE & CLEAR: Store the message and clear input field immediately
    const userMessage = inputMessage.trim();
    inputMessage = '';

    // UPDATE STATE: Add user's message to chat history
    messages = [...messages, { role: 'user', content: userMessage }];

    // SHOW LOADING: Display "AI is thinking..." indicator before streaming starts
    isLoading = true;

    // CREATE PLACEHOLDER: Add empty assistant message that will fill with streamed text
    const assistantMessageIndex = messages.length;
    messages = [...messages, { role: 'assistant', content: '' }];

    try {
      // CLAUDE STREAMING REQUEST: Uses Anthropic's streaming API
      // Claude's API format is slightly different from OpenAI
      const stream = await anthropic.messages.stream({
        model: selectedModel, // Dynamic Claude model selection
        max_tokens: 4096, // Maximum response length (required by Anthropic)
        messages: messages
          .filter(m => m.content) // Filter out empty messages
          .map(m => ({ role: m.role, content: m.content })),
      });

      // PROCESS STREAM: Read tokens as they arrive from Claude
      // Anthropic uses event-based streaming with different event types
      for await (const chunk of stream) {
        // HIDE LOADING: First token received - hide "thinking" indicator
        if (isLoading) {
          isLoading = false;
        }

        // EXTRACT TOKEN: Claude sends tokens in content_block_delta events
        // The text is in chunk.delta.text (different from OpenAI's format)
        if (chunk.type === 'content_block_delta' && chunk.delta?.text) {
          const token = chunk.delta.text;

          // UPDATE MESSAGE: Append new token to the assistant's message
          // This creates the "typing" effect as words appear one by one
          messages = messages.map((msg, idx) =>
            idx === assistantMessageIndex
              ? { ...msg, content: msg.content + token }
              : msg
          );

          // AUTO-SCROLL: Keep chat scrolled to bottom as new text appears
          scrollToBottom();
        }
      }
    } catch (error) {
      // ERROR HANDLING: Show user-friendly error message
      console.error('Error:', error);
      messages = messages.map((msg, idx) =>
        idx === assistantMessageIndex
          ? { ...msg, content: 'Sorry, there was an error. Please check your API key and try again.' }
          : msg
      );
    } finally {
      // CLEANUP: Hide typing indicator and ensure scroll position
      isLoading = false;
      scrollToBottom();
    }
  }

  // SCROLL UTILITY: Smoothly scroll chat to bottom to show latest messages
  function scrollToBottom() {
    setTimeout(() => {
      if (messagesContainer) {
        messagesContainer.scrollTop = messagesContainer.scrollHeight;
      }
    }, 100);
  }

  // KEYBOARD SHORTCUT: Send message when user presses Enter (without Shift)
  // Shift+Enter allows multi-line messages
  function handleKeyPress(event) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault();
      sendMessage();
    }
  }
</script>

<div class="chat-container">
  <div class="messages" bind:this={messagesContainer}>
    {#if messages.length === 0}
      <div class="empty-state">
        <h2>👋 Start a conversation</h2>
        <p>Ask me anything!</p>
      </div>
    {/if}

    {#each messages as message}
      <div class="message {message.role}">
        <div class="message-content">
          <div class="message-header">
            <strong>{message.role === 'user' ? 'You' : 'AI'}</strong>
          </div>
          <div class="message-text">{message.content}</div>
        </div>
      </div>
    {/each}

    <!-- LOADING INDICATOR: Shows animated dots while waiting for first token from AI -->
    <!-- This appears BEFORE streaming starts, then disappears once text begins appearing -->
    {#if isLoading}
      <div class="message assistant">
        <div class="message-content">
          <div class="message-header">
            <strong>AI</strong>
          </div>
          <!-- Three animated dots that bounce up and down -->
          <div class="typing-indicator">
            <span></span>
            <span></span>
            <span></span>
          </div>
        </div>
      </div>
    {/if}
  </div>

  <!-- MODEL SELECTOR: Allows user to choose which AI model to use -->
  <!-- Different models have different capabilities and speeds -->
  <div class="model-selector">
    <label for="model-select">
      <strong>Model:</strong>
    </label>
    <select id="model-select" bind:value={selectedModel} disabled={isLoading}>
      {#each models as model}
        <option value={model.id}>
          {model.name}
        </option>
      {/each}
    </select>
    <!-- DYNAMIC DESCRIPTION: Shows info about currently selected model -->
    <span class="model-description">
      {models.find(m => m.id === selectedModel)?.description}
    </span>
  </div>

  <div class="input-container">
    <textarea
      bind:value={inputMessage}
      on:keypress={handleKeyPress}
      placeholder="Type your message..."
      rows="1"
      disabled={isLoading}
    ></textarea>
    <button on:click={sendMessage} disabled={!inputMessage.trim() || isLoading}>
      Send
    </button>
  </div>
</div>

<style>
  .chat-container {
    display: flex;
    flex-direction: column;
    height: 100%;
    flex: 1;
  }

  .messages {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .empty-state {
    text-align: center;
    color: #666;
    margin: auto;
  }

  .empty-state h2 {
    font-size: 28px;
    margin-bottom: 10px;
  }

  .empty-state p {
    font-size: 16px;
    color: #999;
  }

  .message {
    display: flex;
    max-width: 80%;
    animation: slideIn 0.3s ease-out;
  }

  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .message.user {
    align-self: flex-end;
  }

  .message.assistant {
    align-self: flex-start;
  }

  .message-content {
    padding: 12px 16px;
    border-radius: 12px;
    word-wrap: break-word;
  }

  .message.user .message-content {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
  }

  .message.assistant .message-content {
    background: #f0f0f0;
    color: #333;
  }

  .message-header {
    font-size: 12px;
    margin-bottom: 4px;
    opacity: 0.8;
    text-align: left;

  }

  .message-text {
    font-size: 15px;
    line-height: 1.5;
    white-space: pre-wrap;
    text-align: left;
  }

  .typing-indicator {
    display: flex;
    gap: 4px;
    padding: 8px 0;
  }

  .typing-indicator span {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #999;
    animation: typing 1.4s infinite;
  }

  .typing-indicator span:nth-child(2) {
    animation-delay: 0.2s;
  }

  .typing-indicator span:nth-child(3) {
    animation-delay: 0.4s;
  }

  @keyframes typing {
    0%, 60%, 100% {
      opacity: 0.3;
      transform: translateY(0);
    }
    30% {
      opacity: 1;
      transform: translateY(-10px);
    }
  }

  /* MODEL SELECTOR STYLES: Dropdown to choose AI model */
  .model-selector {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 12px 20px;
    background: #f9f9f9;
    border-top: 1px solid #e0e0e0;
    border-bottom: 1px solid #e0e0e0;
    font-size: 14px;
  }

  .model-selector label {
    color: #666;
    white-space: nowrap;
  }

  .model-selector select {
    padding: 8px 12px;
    border: 2px solid #e0e0e0;
    border-radius: 6px;
    font-size: 14px;
    font-family: inherit;
    background: white;
    cursor: pointer;
    outline: none;
    transition: border-color 0.2s;
    min-width: 180px;
  }

  .model-selector select:hover:not(:disabled) {
    border-color: #667eea;
  }

  .model-selector select:focus {
    border-color: #667eea;
  }

  .model-selector select:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  /* MODEL DESCRIPTION: Shows helpful info about selected model */
  .model-description {
    color: #666;
    font-size: 13px;
    font-style: italic;
    flex: 1;
  }

  .input-container {
    display: flex;
    gap: 12px;
    padding: 20px;
    border-top: 1px solid #e0e0e0;
    background: white;
  }

  textarea {
    flex: 1;
    padding: 12px 16px;
    border: 2px solid #e0e0e0;
    border-radius: 8px;
    font-size: 15px;
    font-family: inherit;
    resize: none;
    outline: none;
    transition: border-color 0.2s;
    min-height: 24px;
    max-height: 120px;
  }

  textarea:focus {
    border-color: #667eea;
  }

  textarea:disabled {
    background: #f5f5f5;
    cursor: not-allowed;
  }

  button {
    padding: 12px 24px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.2s, transform 0.1s;
  }

  button:hover:not(:disabled) {
    opacity: 0.9;
    transform: translateY(-1px);
  }

  button:active:not(:disabled) {
    transform: translateY(0);
  }

  button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
