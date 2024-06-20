# Ask My Book Client

The React interface for the [Ask My Book prototype](../README.md). It submits questions to the Rails API, renders answers with a typewriter effect, updates the browser path with the stored question ID, and optionally reads responses through browser speech synthesis.

## Setup

```bash
npm install
npm start
```

The current API URL is defined directly in `src/App.js` as `http://localhost:3001`. Start the Rails service on that port before using the client.

## Test command

```bash
npm test
```

See the root README for the retrieval workflow, provider requirements, setup sequence, and project boundaries.
