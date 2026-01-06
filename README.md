
<html>
<head>
    <meta charset="UTF-8">
    <title>AI Flashcard Generator - Technical Documentation</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            max-width: 800px;
            margin: 40px auto;
            padding: 20px;
            line-height: 1.6;
            color: #333;
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #3498db;
            padding-bottom: 10px;
        }
        h2 {
            color: #34495e;
            border-bottom: 2px solid #ecf0f1;
            padding-bottom: 8px;
            margin-top: 30px;
        }
        h3 {
            color: #7f8c8d;
        }
        table {
            border-collapse: collapse;
            width: 100%;
            margin: 15px 0;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: left;
        }
        th {
            background-color: #3498db;
            color: white;
        }
        tr:nth-child(even) {
            background-color: #f9f9f9;
        }
        code {
            background-color: #f4f4f4;
            padding: 2px 6px;
            border-radius: 3px;
            font-family: 'Consolas', monospace;
        }
        pre {
            background-color: #2c3e50;
            color: #ecf0f1;
            padding: 15px;
            border-radius: 5px;
            overflow-x: auto;
        }
        pre code {
            background-color: transparent;
            color: inherit;
        }
        hr {
            border: none;
            border-top: 2px solid #ecf0f1;
            margin: 30px 0;
        }
        strong {
            color: #2c3e50;
        }
        @media print {
            body {
                margin: 0;
                padding: 20px;
            }
        }
    </style>
</head>
<body>
<h1>AI Flashcard Generator</h1>
<h2>Technical Documentation</h2>
<p><strong>Author:</strong> Thiyagarajan Varadharajan<br />
<strong>Repository:</strong> https://github.com/thiyagu26v/ai-flashcard-generator<br />
<strong>Date:</strong> January 2026</p>
<hr />
<h2>1. Problem Understanding &amp; Assumptions</h2>
<h3>1.1 Interpretation</h3>
<p>The task requires building a REST API service that:
- Implements exactly <strong>4 endpoints</strong> (POST, GET, PUT, DELETE)
- Uses <strong>PostgreSQL</strong> for persistent data storage
- Integrates with an <strong>external LLM API</strong> for AI content generation
- Applies <strong>strict validation</strong> using Pydantic
- Includes comprehensive <strong>testing</strong> with Pytest</p>
<h3>1.2 Use Case Selected</h3>
<p>I chose to build an <strong>AI Flashcard Generator</strong> for the following reasons:
- <strong>Educational Relevance</strong>: Flashcards are widely used by students for learning
- <strong>Clear AI Integration</strong>: LLM generates meaningful question-answer pairs from topics
- <strong>Intuitive CRUD Operations</strong>: Flashcard operations are simple to understand and test</p>
<h3>1.3 Assumptions Made</h3>
<table>
<thead>
<tr>
<th>Category</th>
<th>Assumption</th>
<th>Rationale</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Data Formats</strong></td>
<td>Topic: 2-255 characters, Difficulty: enum (easy/medium/hard)</td>
<td>Ensures clean, validated input</td>
</tr>
<tr>
<td><strong>External API</strong></td>
<td>Hugging Face API responds within 60 seconds</td>
<td>Standard timeout for LLM services</td>
</tr>
<tr>
<td><strong>Authentication</strong></td>
<td>No user authentication required</td>
<td>Not specified in requirements</td>
</tr>
<tr>
<td><strong>Rate Limiting</strong></td>
<td>Free tier limits (300 req/hour) acceptable</td>
<td>Demo and evaluation purposes</td>
</tr>
<tr>
<td><strong>Database</strong></td>
<td>PostgreSQL running on localhost:5432</td>
<td>Standard development setup</td>
</tr>
</tbody>
</table>
<p><strong>Ambiguity Resolution:</strong> Chose Hugging Face over other LLM providers because it offers a generous free tier without requiring credit card information.</p>
<hr />
<h2>2. High-Level Design and Architecture</h2>
<h3>2.1 Architecture Overview</h3>
<p>The application follows a <strong>Layered Architecture</strong> pattern:</p>
<pre><code>┌─────────────────────────────────────────────────────────┐
│                    API Layer                            │
│              (FastAPI Routes)                           │
├─────────────────────────────────────────────────────────┤
│                  Service Layer                          │
│     (FlashcardService, LLMService)                      │
├─────────────────────────────────────────────────────────┤
│                   Data Layer                            │
│        (SQLAlchemy Models, PostgreSQL)                  │
└─────────────────────────────────────────────────────────┘
</code></pre>
<h3>2.2 Project Structure</h3>
<pre><code>ai-flashcard-generator/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── config.py            # Configuration management
│   ├── database.py          # Database setup
│   ├── api/routes/          # API endpoints
│   ├── models/              # SQLAlchemy models
│   ├── schemas/             # Pydantic schemas
│   ├── services/            # Business logic
│   └── exceptions/          # Error handling
├── tests/
│   ├── unit/                # Unit tests
│   └── integration/         # API tests
├── .env.example
├── requirements.txt
└── README.md
</code></pre>
<h3>2.3 Key Design Decisions</h3>
<ol>
<li><strong>Async throughout</strong>: All database and HTTP operations are asynchronous</li>
<li><strong>Dependency Injection</strong>: Using FastAPI's <code>Depends()</code> for clean testability</li>
<li><strong>Service Layer Pattern</strong>: Business logic separated from routes</li>
<li><strong>Global Exception Handlers</strong>: Consistent error responses</li>
</ol>
<hr />
<h2>3. Database Schema and External API Integration</h2>
<h3>3.1 Database Schema</h3>
<pre><code>┌─────────────────────────────────────────────────────────┐
│                      flashcards                         │
├─────────────────────────────────────────────────────────┤
│ id          │ INTEGER      │ PRIMARY KEY               │
│ topic       │ VARCHAR(255) │ NOT NULL, INDEXED         │
│ question    │ TEXT         │ NOT NULL (AI-generated)   │
│ answer      │ TEXT         │ NOT NULL (AI-generated)   │
│ difficulty  │ VARCHAR(50)  │ NOT NULL                  │
│ created_at  │ TIMESTAMP    │ NOT NULL                  │
│ updated_at  │ TIMESTAMP    │ NOT NULL                  │
└─────────────────────────────────────────────────────────┘
</code></pre>
<p><strong>Indexing Strategy:</strong>
- Primary key on <code>id</code>
- Index on <code>topic</code> for search optimization
- Composite index on <code>(topic, difficulty)</code> for filtered queries</p>
<h3>3.2 External API Integration</h3>
<p><strong>Provider:</strong> Hugging Face Inference API<br />
<strong>Model:</strong> meta-llama/llama-3.1-8b-instruct</p>
<table>
<thead>
<tr>
<th>Aspect</th>
<th>Implementation</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Authentication</strong></td>
<td>Bearer token via environment variable</td>
</tr>
<tr>
<td><strong>Timeout</strong></td>
<td>60 seconds</td>
</tr>
<tr>
<td><strong>Rate Limit Handling</strong></td>
<td>Detect 429 → Return user-friendly error</td>
</tr>
<tr>
<td><strong>Response Parsing</strong></td>
<td>JSON extraction with regex fallback</td>
</tr>
</tbody>
</table>
<hr />
<h2>4. Data Flow Explanation</h2>
<h3>4.1 Create Flashcard Flow (POST)</h3>
<pre><code>1. Client sends POST request with {topic, difficulty}
         │
         ▼
2. FastAPI receives request
         │
         ▼
3. Pydantic validates input (FlashcardCreate schema)
         │
         ▼
4. FlashcardService.create_flashcard() called
         │
         ▼
5. LLMService calls Hugging Face API
         │
         ▼
6. LLM returns {question, answer}
         │
         ▼
7. Flashcard saved to PostgreSQL
         │
         ▼
8. Return 201 Created with flashcard data
</code></pre>
<h3>4.2 API Endpoints</h3>
<table>
<thead>
<tr>
<th>Method</th>
<th>Endpoint</th>
<th>Description</th>
<th>External API</th>
</tr>
</thead>
<tbody>
<tr>
<td>POST</td>
<td><code>/api/v1/flashcards</code></td>
<td>Create with AI Q&amp;A</td>
<td>✅</td>
</tr>
<tr>
<td>GET</td>
<td><code>/api/v1/flashcards/{id}</code></td>
<td>Retrieve by ID</td>
<td>❌</td>
</tr>
<tr>
<td>PUT</td>
<td><code>/api/v1/flashcards/{id}</code></td>
<td>Update flashcard</td>
<td>✅</td>
</tr>
<tr>
<td>DELETE</td>
<td><code>/api/v1/flashcards/{id}</code></td>
<td>Delete flashcard</td>
<td>❌</td>
</tr>
</tbody>
</table>
<hr />
<h2>5. Error Handling &amp; Resilience Strategy</h2>
<h3>5.1 Exception Hierarchy</h3>
<pre><code>FlashcardException (Base)
├── FlashcardNotFoundError  → 404
├── ValidationError         → 422
├── DatabaseError           → 500
└── LLMAPIError (Base)
    ├── LLMTimeoutError     → 503
    ├── LLMRateLimitError   → 429
    └── LLMParseError       → 500
</code></pre>
<h3>5.2 Global Exception Handlers</h3>
<p>All exceptions are caught by FastAPI global handlers and return consistent JSON:</p>
<pre><code class="language-json">{
    &quot;detail&quot;: &quot;Human-readable message&quot;,
    &quot;error_code&quot;: &quot;MACHINE_READABLE_CODE&quot;
}
</code></pre>
<h3>5.3 Resilience Strategies</h3>
<table>
<thead>
<tr>
<th>Failure Scenario</th>
<th>Handling Approach</th>
</tr>
</thead>
<tbody>
<tr>
<td>LLM API timeout</td>
<td>60s timeout → 503 response</td>
</tr>
<tr>
<td>Rate limit exceeded</td>
<td>Return 429 with retry message</td>
</tr>
<tr>
<td>Invalid LLM response</td>
<td>Regex fallback → 500 if fails</td>
</tr>
<tr>
<td>Database connection failed</td>
<td>Rollback → 500 response</td>
</tr>
</tbody>
</table>
<hr />
<h2>6. Testing Strategy</h2>
<h3>6.1 Unit Tests</h3>
<p>Located in <code>tests/unit/</code>:</p>
<ul>
<li><strong>test_schemas.py</strong>: Validates Pydantic schema constraints</li>
<li>Valid/invalid topic lengths</li>
<li>Difficulty enum validation</li>
<li>
<p>Default value handling</p>
</li>
<li>
<p><strong>test_services.py</strong>: Tests business logic with mocked dependencies</p>
</li>
<li>Prompt building logic</li>
<li>JSON response parsing</li>
<li>CRUD operations</li>
</ul>
<h3>6.2 Integration Tests</h3>
<p>Located in <code>tests/integration/</code>:</p>
<ul>
<li><strong>test_flashcard_api.py</strong>: Tests all 4 endpoints</li>
<li>Success scenarios (201, 200, 204)</li>
<li>Error scenarios (404, 422)</li>
<li>Complete CRUD flow</li>
</ul>
<h3>6.3 Mocking Strategy</h3>
<ul>
<li><strong>External API</strong>: Mocked using <code>unittest.mock.AsyncMock</code></li>
<li><strong>Database</strong>: SQLite in-memory for fast test execution</li>
<li><strong>Fixtures</strong>: Shared via <code>conftest.py</code></li>
</ul>
<hr />
<h2>7. Trade-offs, Limitations, and Improvements</h2>
<h3>7.1 Trade-offs Made</h3>
<table>
<thead>
<tr>
<th>Decision</th>
<th>Trade-off</th>
<th>Reasoning</th>
</tr>
</thead>
<tbody>
<tr>
<td>Hugging Face over OpenAI</td>
<td>Less powerful but free</td>
<td>No payment required for evaluation</td>
</tr>
<tr>
<td>SQLite for tests</td>
<td>Different from prod DB</td>
<td>Faster test execution</td>
</tr>
<tr>
<td>No retry logic</td>
<td>Simpler implementation</td>
<td>Left for user retry</td>
</tr>
</tbody>
</table>
<h3>7.2 Current Limitations</h3>
<ol>
<li><strong>No User Authentication</strong>: Not specified in requirements</li>
<li><strong>No List Endpoint</strong>: Only individual flashcard retrieval</li>
<li><strong>No Caching</strong>: Repeated topics call LLM every time</li>
<li><strong>No Pagination</strong>: Would be needed for list endpoint</li>
</ol>
<h3>7.3 Future Improvements</h3>
<ol>
<li><strong>Redis Caching</strong>: Cache LLM responses for repeated topics</li>
<li><strong>Retry Logic</strong>: Exponential backoff for API failures</li>
<li><strong>JWT Authentication</strong>: Secure user sessions</li>
<li><strong>List Endpoint</strong>: With pagination and filtering</li>
<li><strong>Rate Limiting</strong>: Per-user request limits</li>
<li><strong>Docker Support</strong>: Containerized deployment</li>
</ol>
<hr />
<h2>Conclusion</h2>
<p>This project demonstrates a production-ready REST API with:
- Clean, modular architecture
- Proper async/await usage
- Comprehensive error handling
- External API integration
- Well-structured test suite</p>
<p>The codebase follows Python best practices (PEP 8) and FastAPI conventions, ready for further extension and deployment.</p>
<hr />
<p><strong>Repository:</strong> https://github.com/thiyagu26v/ai-flashcard-generator</p>
<p><strong>Author:</strong> Thiyagarajan Varadharajan</p>
</body>
</html>
