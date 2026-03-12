# FastAPI Request Flow — Complete Simple Internal Lifecycle

## Overview

FastAPI does **not work alone**.  
When a request comes from a browser or client, multiple layers cooperate before your Python function executes.

The real flow is:

```text
Client → Uvicorn → ASGI → Starlette → FastAPI → asyncio → Response back
```

---

# 1. Client sends request

A browser, mobile app, or frontend sends an HTTP request.

Example:

```http
GET /users
```

This request contains:

- URL
- HTTP method
- headers
- optional body

At this moment, your FastAPI code has **not executed yet**.

---

# 2. Uvicorn receives the network request

Uvicorn is the **ASGI server**.

Its job is:

- open a port (example: 8000)
- listen continuously
- accept incoming TCP connections
- parse HTTP packets

Example command:

```bash
uvicorn main:app
```

Uvicorn is the **first actual runtime component touching the outside network**.

Think of it as:

> the gatekeeper of your backend server.

---

# 3. Uvicorn converts request into ASGI format

ASGI means:

**Asynchronous Server Gateway Interface**

ASGI is a standard contract that defines:

how Python async web applications communicate with async servers.

Uvicorn converts raw HTTP data into ASGI structures.

Conceptually:

```text
HTTP request → ASGI message
```

ASGI mainly provides:

- scope
- receive
- send

---

# 4. Starlette receives and routes the request

FastAPI is built on Starlette.

Starlette handles:

- routing
- request object creation
- response object handling
- middleware chain

Starlette checks:

```text
Which endpoint matches this path?
```

Example:

```python
@app.get("/users")
```

If URL is `/users`, Starlette sends execution to that endpoint.

---

# 5. FastAPI executes your endpoint logic

FastAPI adds:

- validation
- dependency injection
- automatic docs
- typed request parsing

Example endpoint:

```python
@app.get("/users")
async def get_users():
    return {"users": []}
```

At this stage your actual business logic runs.

---

# 6. asyncio manages coroutine execution

If endpoint contains:

```python
await something()
```

then Python coroutine pauses.

Example:

```python
await db.fetch()
```

During waiting:

- current coroutine pauses
- event loop schedules another task

This is where concurrency happens.

---

# 7. Event loop resumes paused coroutine

When external operation finishes:

- DB result arrives
- network response arrives

event loop resumes exactly where execution stopped.

Coroutine continues:

```python
return data
```

---

# 8. Response returns back through reverse chain

Response travels backward:

```text
FastAPI → Starlette → Uvicorn → Client
```

Uvicorn converts Python response into HTTP packet.

Browser receives final result.

---

# Internal Responsibility Summary

| Component | Responsibility |
|-----------|----------------|
| Uvicorn | Server, network listening |
| ASGI | Communication standard |
| Starlette | Routing and web core |
| FastAPI | Validation + developer layer |
| asyncio | Coroutine scheduling |

---

# Easy Mental Model

- **Uvicorn = receives traffic**
- **ASGI = communication language**
- **Starlette = route manager**
- **FastAPI = your application logic**
- **asyncio = pause/resume manager**

---

# Final One-Line Flow

```text
Client → Uvicorn → ASGI → Starlette → FastAPI → asyncio → Response
```
