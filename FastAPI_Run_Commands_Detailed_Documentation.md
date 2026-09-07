# Running a FastAPI Python File: `fastapi dev` vs `uvicorn`

## 1. Overview

When you create a FastAPI application in a Python file, there are two common ways to start it during development:

```zsh
fastapi dev server.py
```

and:

```zsh
uvicorn server:app --reload
```

Both can run the same FastAPI application.

The important difference is that:

- `fastapi dev server.py` uses the **FastAPI CLI**.
- `uvicorn server:app --reload` invokes **Uvicorn directly**.
- FastAPI is the **web framework/application**.
- Uvicorn is the **ASGI web server** that runs the application.
- ASGI is the **interface/contract** that allows an ASGI server such as Uvicorn to communicate with an ASGI application such as FastAPI.

Neither approach is deprecated.

---

# 2. A Basic FastAPI Application

Suppose you have a file called:

```text
server.py
```

containing:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello"}
```

There are two important things here:

### `server.py`

This is the Python module/file containing your FastAPI application.

### `app`

This is the FastAPI application object:

```python
app = FastAPI()
```

The two pieces become important when using Uvicorn:

```text
server:app
```

This means:

```text
server  → Python module
app     → object inside that module
```

Conceptually, it is similar to:

```python
from server import app
```

---

# 3. Method 1 — FastAPI CLI

The modern FastAPI CLI command is:

```zsh
fastapi dev server.py
```

Here:

```text
fastapi
   ↓
FastAPI command-line interface

dev
   ↓
Start the application in development mode

server.py
   ↓
The Python file containing the FastAPI application
```

For example:

```zsh
fastapi dev server.py
```

The CLI discovers the FastAPI application and starts a development server.

You will normally get output containing something similar to:

```text
Server started at http://127.0.0.1:8000
```

You can then open:

```text
http://127.0.0.1:8000
```

---

# 4. Why Is `fastapi server.py` Invalid?

You may see an older tutorial showing:

```zsh
fastapi server.py
```

With the current FastAPI CLI, this is not the correct syntax.

The CLI expects a command first.

The modern structure is:

```text
fastapi <command> <file>
```

For development:

```zsh
fastapi dev server.py
```

For running the application:

```zsh
fastapi run server.py
```

So:

```zsh
fastapi server.py
```

is interpreted as trying to execute a command named `server.py`.

That is why you may see an error similar to:

```text
No such command 'server.py'
```

This does **not** mean your Python file is in the wrong location.

The issue is simply the command syntax.

---

# 5. `fastapi dev` vs `fastapi run`

FastAPI CLI provides different modes.

## Development

Use:

```zsh
fastapi dev server.py
```

This is intended for local development.

It provides development-friendly behavior such as automatic reloading when your source code changes.

## Running the application

Use:

```zsh
fastapi run server.py
```

This is intended for running the application rather than using the development workflow.

A simple way to remember this is:

```text
fastapi dev
    ↓
I am developing the application

fastapi run
    ↓
I want to run the application
```

---

# 6. Method 2 — Run Uvicorn Directly

You can also bypass the FastAPI CLI and run Uvicorn directly:

```zsh
uvicorn server:app --reload
```

This is a completely valid and commonly used way to run FastAPI applications.

It is **not deprecated**.

The syntax is:

```text
uvicorn <module>:<application_object>
```

Therefore:

```zsh
uvicorn server:app
```

means:

```text
server
  ↓
server.py

app
  ↓
FastAPI object inside server.py
```

---

# 7. Understanding `server:app`

This is one of the most important concepts to understand.

Suppose your project is:

```text
project/
│
└── server.py
```

And `server.py` contains:

```python
from fastapi import FastAPI

app = FastAPI()
```

Then:

```zsh
uvicorn server:app
```

means:

```text
server
  │
  └── server.py

app
  │
  └── FastAPI application object
```

The colon is separating the:

```text
MODULE : OBJECT
```

So:

```text
server:app
```

means:

```text
MODULE = server
OBJECT = app
```

---

# 8. Why Don't We Write `server.py` with Uvicorn?

With Uvicorn, you normally do **not** write:

```zsh
uvicorn server.py:app
```

Instead, you write:

```zsh
uvicorn server:app
```

Why?

Because Uvicorn expects a Python **import string**:

```text
<module>:<attribute>
```

Python modules are generally referred to without the `.py` extension.

For example:

```text
server.py
```

becomes:

```text
server
```

Therefore:

```zsh
uvicorn server:app
```

is correct.

---

# 9. What Does `--reload` Do?

You will frequently see:

```zsh
uvicorn server:app --reload
```

The:

```text
--reload
```

option enables automatic reloading during development.

For example, suppose your application contains:

```python
@app.get("/hello")
def hello():
    return {"message": "Hello"}
```

You start:

```zsh
uvicorn server:app --reload
```

Then you modify the response:

```python
return {"message": "Hello World"}
```

Uvicorn detects the source-code change and restarts the development server.

Without `--reload`, you would normally have to stop and restart the server manually.

Therefore:

```zsh
uvicorn server:app
```

means:

```text
Run the application
```

while:

```zsh
uvicorn server:app --reload
```

means:

```text
Run the application
and automatically restart it when code changes
```

`--reload` is mainly intended for development, not production.

---

# 10. What Actually Happens When You Run Uvicorn?

Consider:

```zsh
uvicorn server:app --reload
```

A simplified sequence is:

```text
Terminal
   │
   │ uvicorn server:app --reload
   ▼
Uvicorn
   │
   │ imports server.py
   ▼
server.py
   │
   │ finds app
   ▼
app = FastAPI()
   │
   ▼
Uvicorn runs the FastAPI application
```

Uvicorn becomes the server that listens for HTTP requests.

---

# 11. What Is a Web Server?

A web server is a program that listens for incoming network requests.

For example, your browser requests:

```text
GET /hello
```

The server receives that request and eventually sends back something such as:

```json
{
    "message": "Hello"
}
```

In a FastAPI application, Uvicorn performs the server-side network work.

A simplified model is:

```text
Browser
   │
   │ HTTP request
   ▼
Uvicorn
   │
   │ ASGI communication
   ▼
FastAPI
   │
   │ calls your function
   ▼
hello()
```

---

# 12. What Is FastAPI?

FastAPI is a Python web framework.

It helps you build APIs using Python.

For example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello"}
```

FastAPI is responsible for framework-level functionality such as:

- URL routing
- Request handling
- Data validation
- Dependency injection
- Serialization
- OpenAPI schema generation
- Interactive API documentation

Your function:

```python
def hello():
    return {"message": "Hello"}
```

contains the application logic for that endpoint.

---

# 13. What Is Uvicorn?

Uvicorn is an **ASGI web server**.

Its job is primarily to:

1. Listen for network connections.
2. Receive HTTP requests.
3. Communicate with the FastAPI application through ASGI.
4. Receive the application's response.
5. Send the HTTP response back to the client.

So Uvicorn is not the same thing as FastAPI.

Think:

```text
FastAPI = application/framework

Uvicorn = server that runs the application
```

---

# 14. What Is ASGI?

ASGI stands for:

**Asynchronous Server Gateway Interface**

The easiest way to understand ASGI is as a **standard interface/contract** between an ASGI server and an ASGI application.

It defines how the server and application communicate.

It is better to think of ASGI as a communication contract than as a simple data format like JSON.

The architecture looks like:

```text
Client
  │
  │ HTTP
  ▼
Uvicorn
  │
  │ ASGI
  ▼
FastAPI
```

Uvicorn does not need to know the internal implementation of FastAPI.

It only needs to know how to communicate with an ASGI application.

---

# 15. FastAPI, Uvicorn and ASGI Together

The relationship can be summarized as:

```text
                 ASGI
        communication contract
              ↕
┌───────────────────────────┐
│         Uvicorn           │
│      ASGI Server          │
└───────────────────────────┘
              ↕
┌───────────────────────────┐
│         FastAPI            │
│      ASGI Application      │
└───────────────────────────┘
```

More practically:

```text
Browser
   │
   │ HTTP request
   ▼
Uvicorn
   │
   │ ASGI communication
   ▼
FastAPI
   │
   │ route matching
   ▼
Your Python function
```

---

# 16. Complete Request Flow

Suppose the browser requests:

```text
GET http://127.0.0.1:8000/hello
```

The flow is approximately:

```text
┌──────────────┐
│   Browser    │
└──────┬───────┘
       │
       │ HTTP GET /hello
       ▼
┌──────────────┐
│   Uvicorn    │
│  ASGI Server │
└──────┬───────┘
       │
       │ ASGI
       ▼
┌──────────────┐
│   FastAPI    │
└──────┬───────┘
       │
       │ Find /hello route
       ▼
┌──────────────┐
│ hello()      │
└──────┬───────┘
       │
       │ {"message": "Hello"}
       ▼
┌──────────────┐
│   FastAPI    │
└──────┬───────┘
       │
       │ ASGI response
       ▼
┌──────────────┐
│   Uvicorn    │
└──────┬───────┘
       │
       │ HTTP response
       ▼
┌──────────────┐
│   Browser    │
└──────────────┘
```

---

# 17. Response Flow

The response travels back in the opposite direction.

Your function:

```python
def hello():
    return {"message": "Hello"}
```

returns a Python dictionary.

FastAPI processes that result and prepares the response.

Conceptually:

```text
hello()
   │
   ▼
FastAPI
   │
   │ ASGI response information
   ▼
Uvicorn
   │
   │ HTTP response
   ▼
Browser
```

The browser ultimately receives an HTTP response.

---

# 18. Is FastAPI Sending an "ASGI Format"?

You may hear an explanation like:

> FastAPI sends ASGI format to Uvicorn.

This is useful as a beginner mental model, but technically it is more accurate to say:

> FastAPI and Uvicorn communicate through the ASGI interface.

ASGI defines the expected communication between:

```text
ASGI Server
      ↕
ASGI Application
```

Uvicorn is the ASGI server.

FastAPI is an ASGI application.

---

# 19. Does FastAPI Only Work With Uvicorn?

No.

FastAPI is an ASGI application, so it can work with compatible ASGI servers.

Examples include:

- Uvicorn
- Hypercorn
- Daphne
- Granian

Uvicorn is one of the most commonly used choices.

This is possible because FastAPI follows the ASGI interface.

Conceptually:

```text
             ASGI
              │
      ┌───────┼────────┐
      │       │        │
  Uvicorn  Hypercorn  Daphne
      │
      ▼
   FastAPI
```

---

# 20. Why Does FastAPI Have Its Own CLI If Uvicorn Already Exists?

The FastAPI CLI provides a more convenient developer experience.

You can write:

```zsh
fastapi dev server.py
```

instead of explicitly writing:

```zsh
uvicorn server:app --reload
```

The FastAPI CLI handles application discovery and starts the server for you.

A simplified mental model is:

```text
fastapi dev server.py
          │
          ▼
     FastAPI CLI
          │
          ▼
       Uvicorn
          │
          ▼
       FastAPI
```

So the FastAPI CLI does not mean that Uvicorn has disappeared.

Uvicorn is still the server underneath the development setup.

---

# 21. FastAPI CLI vs Uvicorn Directly

| Feature | FastAPI CLI | Uvicorn |
|---|---|---|
| Example | `fastapi dev server.py` | `uvicorn server:app --reload` |
| Who starts the server? | FastAPI CLI | You directly invoke Uvicorn |
| Need module/object syntax? | Usually no | Yes |
| Example module syntax | `server.py` | `server:app` |
| Development reload | Built into `dev` workflow | `--reload` |
| Valid today? | Yes | Yes |
| Deprecated? | No | No |

---

# 22. Which Command Should You Use?

For learning FastAPI, both are useful.

### Simple FastAPI development

Use:

```zsh
fastapi dev server.py
```

This is convenient and easy to remember.

### Learning how FastAPI actually runs

Also understand:

```zsh
uvicorn server:app --reload
```

This is especially valuable because it makes the architecture clearer:

```text
server.py
   ↓
server
   ↓
app
   ↓
Uvicorn
   ↓
ASGI
   ↓
FastAPI
```

Understanding the Uvicorn command is important even if you mostly use the FastAPI CLI.

---

# 23. Common Command Mistakes

## Mistake 1

```zsh
fastapi server.py
```

### Problem

The current FastAPI CLI expects a command such as `dev` or `run`.

### Correct

```zsh
fastapi dev server.py
```

or:

```zsh
fastapi run server.py
```

---

## Mistake 2

```zsh
uvicorn server.py:app
```

### Problem

Uvicorn expects a Python module name, not the filename with `.py`.

### Correct

```zsh
uvicorn server:app
```

---

## Mistake 3

Suppose the file contains:

```python
my_app = FastAPI()
```

but you run:

```zsh
uvicorn server:app
```

### Problem

There is no `app` object in `server.py`.

### Correct

```zsh
uvicorn server:my_app
```

The second part must match the actual object name.

---

# 24. Example With a Different Filename

Suppose your file is:

```text
main.py
```

and it contains:

```python
from fastapi import FastAPI

application = FastAPI()

@application.get("/")
def home():
    return {"message": "Welcome"}
```

You could run it directly with Uvicorn using:

```zsh
uvicorn main:application --reload
```

Breakdown:

```text
main
  ↓
main.py

application
  ↓
application = FastAPI()
```

The same pattern applies regardless of the filename.

---

# 25. Example Project Structure

A small FastAPI project might look like:

```text
my_project/
│
├── server.py
├── requirements.txt
└── ...
```

`server.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Welcome to FastAPI"}
```

From the project directory:

```zsh
uvicorn server:app --reload
```

or:

```zsh
fastapi dev server.py
```

---

# 26. Why Does the Terminal Need to Be in the Project Directory?

When you run:

```zsh
uvicorn server:app
```

Uvicorn needs to be able to import the `server` module.

If you are inside:

```text
my_project/
```

and have:

```text
my_project/server.py
```

Python can normally import:

```python
import server
```

Therefore:

```zsh
uvicorn server:app
```

works.

If you run the command from an unrelated directory, Python may not be able to find the module.

You can think of it as:

```text
Current directory
       │
       ▼
Python import path
       │
       ▼
server.py
```

This is one reason project structure and the current working directory matter.

---

# 27. Using a Package Structure

As projects become larger, you may have:

```text
my_project/
│
└── app/
    ├── __init__.py
    └── main.py
```

If `main.py` contains:

```python
from fastapi import FastAPI

app = FastAPI()
```

you can run:

```zsh
uvicorn app.main:app --reload
```

The syntax becomes:

```text
app.main:app
│       │
│       └── object
│
└── Python module
```

More specifically:

```text
app
 ↓
package

main
 ↓
main.py

app
 ↓
FastAPI object
```

---

# 28. The Most Important Mental Model

Remember this:

```text
                 CLIENT
                   │
                   │ HTTP
                   ▼
              ┌─────────┐
              │ Uvicorn │
              └────┬────┘
                   │
                  ASGI
                   │
                   ▼
              ┌─────────┐
              │ FastAPI │
              └────┬────┘
                   │
             Route matching
                   │
                   ▼
             Your function
```

And the command:

```zsh
uvicorn server:app --reload
```

means:

```text
Uvicorn
  ↓
Find/import server module
  ↓
Find app object
  ↓
Run it as an ASGI application
  ↓
Reload it when code changes
```

---

# 29. Quick Command Cheat Sheet

## Start FastAPI development server

```zsh
fastapi dev server.py
```

## Start FastAPI application

```zsh
fastapi run server.py
```

## Start Uvicorn directly

```zsh
uvicorn server:app
```

## Start Uvicorn with automatic reload

```zsh
uvicorn server:app --reload
```

## Different filename

If you have:

```text
main.py
```

and:

```python
app = FastAPI()
```

use:

```zsh
uvicorn main:app --reload
```

## Different application object

If you have:

```python
application = FastAPI()
```

use:

```zsh
uvicorn main:application --reload
```

---

# 30. Final Summary

There are three concepts to keep separate:

### FastAPI

The Python web framework/application.

```text
FastAPI
```

### Uvicorn

The ASGI server that runs the application and handles the network/HTTP side.

```text
Uvicorn
```

### ASGI

The standard interface/communication contract between the server and the application.

```text
Uvicorn
   ↕
  ASGI
   ↕
FastAPI
```

The two common commands are:

```zsh
fastapi dev server.py
```

and:

```zsh
uvicorn server:app --reload
```

Both are valid.

`fastapi dev server.py` is the convenient FastAPI CLI approach.

`uvicorn server:app --reload` is the direct Uvicorn approach.

And:

```text
server:app
```

means:

```text
server.py → app object
```

Finally, `--reload` means:

```text
Automatically restart the development server
when your code changes.
```

So if you remember only one architecture, remember:

```text
Browser
   │
   │ HTTP
   ▼
Uvicorn
   │
   │ ASGI
   ▼
FastAPI
   │
   ▼
Your Python route function
```
