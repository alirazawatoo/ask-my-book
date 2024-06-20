# Ask My Book

A Rails and React prototype for asking questions about a PDF. The backend extracts page text, creates OpenAI embeddings, retrieves relevant passages, and sends a context-limited prompt to a completion model. The browser client displays the answer progressively and can read it aloud with the Web Speech API.

This is retrieval-augmented question answering over a configured book—not model training or fine-tuning.

## How it works

```text
Configured PDF
     |
     v
Extract and clean page text
     |
     v
Count tokens and create page embeddings
     |
     v
Persist text + embedding CSV files
     |
     +-----------------------------+
                                   |
Question --> query embedding --> similarity ranking
                                   |
                                   v
                          selected page context
                                   |
                                   v
                         completion-model answer
                                   |
                                   v
                     Rails cache + React interface
```

## Implemented workflow

- Parses a configured PDF with `pdf-reader`
- Creates page-level text records and token counts
- Generates and stores document embeddings
- Embeds each incoming question and ranks pages by vector similarity
- Limits selected context before constructing the prompt
- Stores questions, answers, context, and repeat counts
- Returns cached answers for previously stored questions
- Supports shareable question paths in the React client
- Uses browser speech synthesis for answer playback

## API

```http
POST /api/v1/questions
Content-Type: application/json

{
  "title": "How should a small business choose its first product?"
}
```

```json
{
  "question": "How should a small business choose its first product?",
  "answer": "<generated answer>",
  "id": 42
}
```

Supplying an existing `id` retrieves the stored question path and cached response behavior.

## Repository layout

```text
app/
├── controllers/api/v1/questions_controller.rb
├── models/question.rb
└── services/
    ├── load_book.rb          # PDF extraction and document embeddings
    └── handle_questions.rb   # Retrieval, prompt construction, and caching
client/
└── src/App.js                # Question UI, progressive answer, and speech
```

## Local setup

### Prerequisites

- Ruby 3.0.1
- Node.js and npm
- SQLite
- Redis
- An OpenAI API key compatible with the configured legacy model calls
- A PDF you are permitted to process

```bash
git clone https://github.com/alirazawatoo/ask-my-book.git
cd ask-my-book
bundle install
bin/rails db:prepare
bin/rails server -p 3001
```

In another terminal:

```bash
cd client
npm install
npm start
```

The client uses `http://localhost:3001/api/v1/questions` directly. Provider credentials, model names, PDF paths, and generated CSV locations are configured in the Rails initializers and environment.

## Test boundary

The React workspace includes a basic `App.test.js` file. No focused backend retrieval, embedding, prompt-budget, cache, or API tests are tracked, so answer quality and current provider compatibility are not claimed.

## Important boundaries

- Prompt text and examples are tailored to one specific book and author.
- The project uses older completion and embedding interfaces that may require adaptation for a current provider SDK.
- Generated embedding CSVs are local artifacts rather than a managed vector index.
- Similarity uses a raw dot product and assumes compatible embedding vectors.
- The question endpoint does not authenticate callers, validate ownership, or enforce rate limits.
- Book text, example answers, and generated outputs require appropriate content rights before public deployment.

## Next improvements

1. Separate book configuration and prompt examples from application code.
2. Add citations by returning the selected page identifiers with every answer.
3. Cover ingestion, retrieval ranking, prompt budgets, caching, and provider failures with deterministic tests.
4. Move provider calls behind a versioned adapter and use a maintained model API.
5. Add upload validation, authentication, quotas, and a managed retrieval index for multi-book use.
