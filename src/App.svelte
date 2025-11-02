<script>
  import Chat from './lib/Chat.svelte'
</script>

<main>
  <div class="app-container">
    <header>
      <h1>AI Chat</h1>
      <p class="subtitle">Powered by OpenAI</p>
    </header>
    <Chat />
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
    background: #f7f7f8;
  }

  main {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    padding: 20px;
  }

  .app-container {
    width: 100%;
    max-width: 900px;
    height: 90vh;
    display: flex;
    flex-direction: column;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.07);
    overflow: hidden;
  }

  header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 20px 30px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  h1 {
    margin: 0;
    font-size: 24px;
    font-weight: 600;
  }

  .subtitle {
    margin: 5px 0 0 0;
    font-size: 14px;
    opacity: 0.9;
  }
</style>
