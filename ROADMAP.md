# ROADMAP

## General Principles
- One concept per mini-project.
- Projects should take roughly 30–120 minutes.
- Focus on concepts over frameworks.
- Build experiments, not products.

Each README should answer:
1. What problem does this concept solve?
2. How does it work?
3. When would you use it?
4. What did I build?
5. Key takeaways.

## Phase 1: Backend Fundamentals

PROJECT: Library API

1. Project Set-up & Configuration: Configure linting, formatting, gitignore, and separate test / dev / prod environments, variables, and scripts.
- Typescript
- ESLint
- Prettier
- dotenv
- gitignore
- package.json
- tsconfig.json
- config.server.port / config.database.url / config.jwt.secret instead of process.env.PORT

Goal: create the foundation.

Build:
backend-fundamentals/
  src/
  tests/
  .env
  .env.test
  .gitignore
  package.json
  tsconfig.json

Success Criteria:
npm run dev
npm run build
npm run test
npm run lint

Create a typed config module:
config.server.port
config.database.url
config.jwt.secret

2. HTTP Server: Build a web server using Node's http module.
- Node.js http
- Postman

Build a raw Node server with:
GET /
GET /health
GET /books
POST /books
DELETE /books/:id
Use an in-memory array.

Success criteria:
Postman can create, list, and delete books.

3. REST APIs: Build same web server using Express.
- Express

Goal: understand what Express abstracts.

Rebuild the same API using Express:
GET /health
GET /books
GET /books/:id
POST /books
PUT /books/:id
DELETE /books/:id
Still use an in-memory array.

Success criteria:
Same behavior as raw Node version, but cleaner routes.

4. Routing: Support /users, /books, /orders cleanly.

Goal: organize routes cleanly.

Add route groups:
/books
/users
/orders or /loans

Suggested domain:
Users borrow Books through Loans

Routes:
GET /users
POST /users
GET /books
POST /books
GET /loans
POST /loans

Success criteria:
No giant app.ts. Each route has its own file.

5. Pagination & Filtering

Goal: learn common list endpoint patterns.

Add to GET /books:
?page=1&limit=10
?search=dune
?author=asimov
?sort=title
?available=true

Success criteria:
{
  "data": [],
  "page": 1,
  "limit": 10,
  "total": 42
}

6. SQL: Replace your in-memory array with PostgreSQL.
- Prisma
- PostgreSQL

Goal: replace memory with real persistence.
Add PostgreSQL + Prisma.

Models:
User
Book
Loan

Build:
GET /books
POST /books
GET /users
POST /users
POST /loans

Success criteria:
Restarting the server does not erase data.

7. Database Migrations

Goal: learn schema evolution.

Start with:
Book: title, author

Then add:
isbn
publishedYear
availableCopies

Create and run migrations.

Success criteria:
Database schema changes without manually editing the database.

8. Validation
- Zod

Goal: validate input before it reaches your business logic.

Add Zod schemas for:
CreateBookInput
UpdateBookInput
CreateUserInput
CreateLoanInput

Success criteria:
Bad requests return 400 with useful validation errors.

Example:
{
  "error": "Invalid request body",
  "details": [...]
}

9. Testing
- Vitest
- Supertest

Goal: prove your API works.

Use Vitest + Supertest.

Write tests for:
GET /health
POST /books
GET /books
POST /books with invalid body
POST /users
POST /loans

Success criteria:
npm test

runs without manually starting the server.

10. Middleware: Implement error handler, CORS, and request validators.
- express.json()
- CORS
- request validation
- error handling

Goal: understand request pipeline.

Implement middleware for:
express.json()
CORS
request validation
central error handling
not-found handler

Success criteria:
Your routes are cleaner because shared behavior moved into middleware.

11. Logging & Observability
- Pino
- request IDs
- structured logs

Goal: understand what happened during a request.

Add Pino.

Every request should log:
requestId
method
path
statusCode
durationMs

Success criteria:
A failed request can be traced through logs using one request ID.

12. Cache
- Redis

Goal: avoid unnecessary database reads.

Add Redis caching for:
GET /books
GET /books/:id

Invalidate cache when:
POST /books
PUT /books/:id
DELETE /books/:id

Success criteria:
First request hits database. Second request hits cache.

13. Authentication: Registration, login, sign-in.
- Sessions
- JWT
- bcrypt

Goal: identify the user.

Build:
POST /auth/register
POST /auth/login
POST /auth/logout
GET /auth/me

Use:
bcrypt
sessions or JWT

Success criteria:
A user can register, login, and retrieve their profile.

14. Authorization: Protected endpoints, middleware (requireAuth, roles, permissions).

Goal: restrict actions by user/role.

Add roles:
ADMIN
MEMBER

Rules:
Anyone can view books.
Only ADMIN can create/update/delete books.
Only logged-in users can create loans.
Users can only view their own loans.

Success criteria:
Unauthorized requests return 401; forbidden requests return 403.

15. Object Storage
- Multer
- Amazon S3

Goal: handle uploaded files.

Add book cover uploads.

Build:
POST /books/:id/cover
GET /books/:id/cover

Use:
Multer
S3 or local S3-compatible storage

Store metadata in PostgreSQL:
fileName
mimeType
size
storageKey
bookId

Success criteria:
Uploading a cover attaches it to a book.

16. Background Jobs: Worker processes uploads asynchronously.
- BullMQ
- asynchronous processing

Goal: move slow work out of the request cycle.

When a cover is uploaded:
API accepts upload
returns 202 Accepted
worker processes image/metadata
updates database

Use BullMQ.

Success criteria:
The API responds immediately while a worker handles processing.

17. Queue Patterns: Retries, dead-letter queues, delayed jobs, scheduled jobs, idempotent workers.

Goal: understand robust job processing.

Enhance the background job system with:
retries
delayed jobs
failed jobs
dead-letter behavior
idempotent workers

Example job:
Send loan reminder email 7 days after book checkout.

Success criteria:
Failed jobs retry safely and do not duplicate side effects.

18. Webhooks

Goal: receive events from external systems.

Create:
POST /webhooks/payment

Fake external event:
{
  "eventId": "evt_123",
  "type": "payment.succeeded",
  "userId": "abc"
}

Success criteria:
Webhook events are stored, processed once, and duplicate events are ignored.

19. API Versioning

Goal: support API changes without breaking old clients.

Create:
/api/v1/books
/api/v2/books

Difference:
v1 returns title + author
v2 returns title + author + availabilityStatus

Success criteria:
Both versions work at the same time.

20. Idempotency

Goal: make retries safe.

Add idempotency keys to:
POST /loans
POST /webhooks/payment

Header:
Idempotency-Key: abc-123

Success criteria:
Sending the same request twice does not create two loans or process two payments.

21. Docker: Containerize the application.

Goal: run the app consistently anywhere.

Build a Dockerfile for the API.

Success criteria:
docker build .
docker run ...

starts the API.

22. Docker Compose: Run API + PostgreSQL + Redis together.

Goal: run the whole backend stack locally.

Compose services:
api
postgres
redis
worker

Success criteria:
docker compose up

starts the full system.

23. Deployment: Deploy API, worker, database, Redis, and environment variables to a cloud platform.

Goal: run your backend outside your laptop.

Build:
Deploy the Library API to a cloud platform.

Start simple:
Render / Railway / Fly.io

Deploy the complete Library API with:
- public API URL
- hosted PostgreSQL
- hosted Redis
- background worker
- object storage
- production environment variables
- /health endpoint
- logs visible in dashboard

Deploy:
api
worker
postgres
redis
environment variables

Success criteria:
Public API URL works
Database persists
Worker runs
Redis connects
Logs are visible
Health check passes

24. API Documentation
- OpenAPI
- Swagger

Goal: document the API like a real service.

Add OpenAPI/Swagger docs for:
/auth
/users
/books
/loans
/webhooks

Success criteria:
You can open Swagger UI and test the API from the browser.

## Future Phases

Phase 2 – AI Engineering

1. Tokenization – Inspect tokenization.
2. Embeddings – Compare document similarity.
3. Cosine Similarity – Implement yourself.
4. Chunking – Split a large document and search it.
5. Vector Search – Semantic search over documents.
6. Retrieval-Augmented Generation (RAG).
7. Function Calling – Weather/calculator/search tools.
8. Structured Output – Force validated JSON.
9. Streaming – Stream generated tokens.
10. Model Serving – Expose an LLM through POST /chat.
11. Prompt Caching.
12. Model Evaluation.
13. Prompt Versioning.
14. Batch Inference.
15. GPU Utilization Basics.
16. Guardrails & Safety Evaluation.

Phase 3 – Distributed Systems

1. Load Balancer – Alternate requests between servers.
2. Caching – Redis in front of the database.
3. Retry Logic – Retry transient failures.
4. Rate Limiting – Requests/minute per client.
5. Circuit Breaker – Stop calling failing services.
6. Pub/Sub – Simple chat room.
7. Eventual Consistency – Two databases syncing periodically.
8. Replication – Write to two servers.
9. Sharding – Partition users across databases.
10. Distributed Tracing – Track requests across services.
11. Distributed Locks.
12. Leader Election.
13. Service Discovery.
14. Health Checks.
15. Graceful Shutdown.
16. Autoscaling.
17. Consistent Hashing.

Phase 4 – AI Agents

1. Tool Calling
2. Memory
3. Reflection
4. Planning
5. Multi-Agent Systems
6. Workflow Graphs
7. Human Approval
8. Long-Term Memory

Phase 5 – LLM Internals

1. Attention
2. Transformer Forward Pass
3. Sampling (temperature, top-k, top-p)
4. Context Window
5. KV Cache
6. Quantization

Phase 6 – Model Training

1. Gradient Descent
2. Small Neural Network (NumPy)
3. Fine-Tuning
4. LoRA
5. Distributed Training