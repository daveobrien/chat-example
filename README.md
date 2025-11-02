# AI Chat Interface

A Claude-like chat interface built with Svelte and powered by OpenAI.

## Features

- Clean, modern chat UI inspired by Claude
- Real-time messaging with OpenAI's GPT models
- Typing indicators
- Smooth animations
- Responsive design

## Setup

### 1. Install Dependencies

```bash
npm install
```

### 2. Add Your OpenAI API Key

Create a `.env` file in the root directory:

```bash
VITE_OPENAI_API_KEY=your_openai_api_key_here
```

Or update the API key directly in `src/lib/Chat.svelte` (line 9)

**Get your API key:** https://platform.openai.com/api-keys

### 3. Run the Development Server

```bash
npm run dev
```

Open `http://localhost:5173` in your browser.

## Important Security Note

⚠️ **This setup uses `dangerouslyAllowBrowser: true`** which means the API key is exposed in the browser. This is **only for development/testing**.

**For production:**
- Create a backend API endpoint
- Store your API key securely on the server
- Make requests through your backend

## Project Structure

```
src/
├── App.svelte           # Main app container
├── lib/
│   └── Chat.svelte      # Chat component with OpenAI integration
├── main.js              # App entry point
└── app.css              # Global styles
```

## Customization

You can easily customize:
- **Model:** Change `gpt-3.5-turbo` to `gpt-4` in Chat.svelte:28
- **Colors:** Update gradient colors in the styles
- **Layout:** Modify the max-width, height in App.svelte
- **Messages:** Add system prompts or conversation history

## Tech Stack

- **Svelte** - Reactive UI framework
- **Vite** - Fast build tool
- **OpenAI SDK** - For GPT integration
- **CSS** - Custom styling

## Next Steps

Some ideas for enhancement:
- Add conversation history persistence (localStorage)
- Implement markdown rendering for responses
- Add code syntax highlighting
- Support for file uploads
- Conversation threads/sessions
- Streaming responses

---

Built with Svelte + OpenAI
