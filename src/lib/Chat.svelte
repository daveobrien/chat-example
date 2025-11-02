<script>
  import OpenAI from 'openai';

  let messages = [];
  let inputMessage = '';
  let isLoading = false;
  let messagesContainer;

  // You'll need to add your OpenAI API key here or via environment variable
  const openai = new OpenAI({
    apiKey: import.meta.env.VITE_OPENAI_API_KEY || 'YOUR_API_KEY_HERE',
    dangerouslyAllowBrowser: true // Note: In production, use a backend server
  });

  async function sendMessage() {
    if (!inputMessage.trim() || isLoading) return;

    const userMessage = inputMessage.trim();
    inputMessage = '';

    // Add user message
    messages = [...messages, { role: 'user', content: userMessage }];
    isLoading = true;

    try {
      const response = await openai.chat.completions.create({
        model: 'gpt-3.5-turbo',
        messages: messages.map(m => ({ role: m.role, content: m.content })),
      });

      const assistantMessage = response.choices[0].message.content;
      messages = [...messages, { role: 'assistant', content: assistantMessage }];
    } catch (error) {
      console.error('Error:', error);
      messages = [...messages, {
        role: 'assistant',
        content: 'Sorry, there was an error. Please check your API key and try again.'
      }];
    } finally {
      isLoading = false;
      scrollToBottom();
    }
  }

  function scrollToBottom() {
    setTimeout(() => {
      if (messagesContainer) {
        messagesContainer.scrollTop = messagesContainer.scrollHeight;
      }
    }, 100);
  }

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

    {#if isLoading}
      <div class="message assistant">
        <div class="message-content">
          <div class="message-header">
            <strong>AI</strong>
          </div>
          <div class="typing-indicator">
            <span></span>
            <span></span>
            <span></span>
          </div>
        </div>
      </div>
    {/if}
  </div>

  <div class="input-container">
    <textarea
      bind:value={inputMessage}
      on:keypress={handleKeyPress}
      placeholder="Type your message..."
      rows="1"
      disabled={isLoading}
    />
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
  }

  .message-text {
    font-size: 15px;
    line-height: 1.5;
    white-space: pre-wrap;
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
