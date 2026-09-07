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

Yes. One small correction first: you mean **Uvicorn**, not Unicorn. 🙂

Here is a Markdown-compatible continuation you can append to your existing document:

# 31. Running FastAPI by Importing Uvicorn in `server.py`

There is another way to run a FastAPI application that is different from both:

```zsh
fastapi dev server.py
```

and:

```zsh
uvicorn server:app --reload
```

You can start Uvicorn **directly from Python code**.

For example:

```python
from fastapi import FastAPI
import uvicorn

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello"}

if __name__ == "__main__":
    uvicorn.run(app)
```

Then you run:

```zsh
python server.py
```

Here, Python executes `server.py`, and the code inside the file starts Uvicorn.

---

# 32. What Is Happening in `python server.py`?

When you execute:

```zsh
python server.py
```

Python runs the file from top to bottom.

Consider:

```python
from fastapi import FastAPI
import uvicorn

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello"}

if __name__ == "__main__":
    uvicorn.run(app)
```

The sequence is:

```text
python server.py
       │
       ▼
Python executes server.py
       │
       ├── import FastAPI
       │
       ├── import uvicorn
       │
       ├── create FastAPI application
       │
       └── uvicorn.run(app)
                  │
                  ▼
              Uvicorn
                  │
                  ▼
              FastAPI app
```

So in this approach, **your Python code explicitly tells Uvicorn to start**.

---

# 33. What Does `uvicorn.run(app)` Mean?

This line:

```python
uvicorn.run(app)
```

tells Uvicorn:

> Start a Uvicorn server and run this FastAPI application.

The `app` here is the actual Python object:

```python
app = FastAPI()
```

So Uvicorn doesn't need to find the application by an import string.

You are directly passing the object to it:

```text
app object
   │
   ▼
uvicorn.run()
   │
   ▼
Uvicorn server
```

---

# 34. Comparing the Three Approaches

There are three common ways to start the same application.

## Approach 1 — FastAPI CLI

```zsh
fastapi dev server.py
```

The FastAPI CLI finds the application and starts the development server.

```text
Terminal
   │
   ▼
FastAPI CLI
   │
   ▼
Uvicorn
   │
   ▼
FastAPI app
```

---

## Approach 2 — Uvicorn CLI

```zsh
uvicorn server:app --reload
```

Here, you directly start Uvicorn from the terminal.

```text
Terminal
   │
   ▼
Uvicorn CLI
   │
   │ imports server
   │ finds app
   ▼
FastAPI app
```

The important part is:

```text
server:app
```

which means:

```text
server.py → app
```

---

## Approach 3 — Uvicorn From Python

```zsh
python server.py
```

Inside `server.py`:

```python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(app)
```

The flow is:

```text
Terminal
   │
   │ python server.py
   ▼
Python
   │
   ▼
server.py
   │
   │ uvicorn.run(app)
   ▼
Uvicorn
   │
   ▼
FastAPI app
```

---

# 35. The Key Difference

The biggest difference is **who starts Uvicorn and how the FastAPI application is supplied to Uvicorn**.

### FastAPI CLI

```zsh
fastapi dev server.py
```

You give the **Python file** to the FastAPI CLI.

```text
server.py
   ↓
FastAPI CLI
   ↓
Uvicorn
   ↓
FastAPI app
```

### Uvicorn CLI

```zsh
uvicorn server:app --reload
```

You give Uvicorn an **import string**:

```text
server:app
```

Uvicorn imports the module and gets the application object.

```text
server:app
   ↓
Uvicorn
   ↓
FastAPI app
```

### Python + `uvicorn.run()`

```zsh
python server.py
```

Your Python code directly passes the application object:

```python
uvicorn.run(app)
```

```text
app object
   ↓
uvicorn.run(app)
   ↓
Uvicorn
```

---

# 36. Import String vs Actual Object

This distinction is very important.

With:

```zsh
uvicorn server:app
```

you are essentially telling Uvicorn:

> Go and import the `server` module, then find the `app` object inside it.

Conceptually:

```python
from server import app
```

With:

```python
uvicorn.run(app)
```

you are saying:

> Here is the actual application object. Run it.

So:

```text
uvicorn server:app
```

uses an **import string**.

Whereas:

```python
uvicorn.run(app)
```

uses the **actual Python object**.

---

# 37. Why Use `if __name__ == "__main__"`?

You will normally see:

```python
if __name__ == "__main__":
    uvicorn.run(app)
```

rather than simply:

```python
uvicorn.run(app)
```

This is important because Python files can be either:

1. Executed directly, or
2. Imported by another Python file.

When you execute:

```zsh
python server.py
```

Python sets:

```python
__name__ = "__main__"
```

Therefore:

```python
if __name__ == "__main__":
```

becomes true and Uvicorn starts.

But if another file does:

```python
import server
```

then:

```python
__name__
```

inside `server.py` is:

```text
server
```

not:

```text
__main__
```

Therefore:

```python
uvicorn.run(app)
```

doesn't automatically execute.

This prevents the server from unintentionally starting whenever the module is imported.

---

# 38. Adding `--reload` Programmatically

You can also enable reload from Python:

```python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        app,
        reload=True
    )
```

Then:

```zsh
python server.py
```

starts Uvicorn with reload enabled.

Conceptually this is similar to:

```zsh
uvicorn server:app --reload
```

but the configuration is being supplied through Python instead of command-line arguments.

---

# 39. Adding Host and Port

You can configure Uvicorn from Python as well.

For example:

```python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        app,
        host="127.0.0.1",
        port=8000
    )
```

Then run:

```zsh
python server.py
```

The equivalent command-line approach would be:

```zsh
uvicorn server:app --host 127.0.0.1 --port 8000
```

So Uvicorn provides configuration through both:

```text
Command-line arguments
```

and:

```text
Python function arguments
```

---

# 40. Side-by-Side Comparison

Suppose we have:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello"}
```

### Option A

```zsh
fastapi dev server.py
```

The FastAPI CLI handles starting the server.

---

### Option B

```zsh
uvicorn server:app --reload
```

Uvicorn is started directly from the terminal.

`server:app` tells Uvicorn where the application is.

---

### Option C

`server.py` contains:

```python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(app, reload=True)
```

Then:

```zsh
python server.py
```

Python starts the file, and the file starts Uvicorn.

---

# 41. Visual Comparison

```text
                OPTION A
        fastapi dev server.py
                    │
                    ▼
             FastAPI CLI
                    │
                    ▼
                Uvicorn
                    │
                    ▼
              FastAPI app
```

```text
                OPTION B
        uvicorn server:app
                    │
                    ▼
                Uvicorn
                    │
              imports server
                    │
              finds app
                    ▼
              FastAPI app
```

```text
                OPTION C
          python server.py
                    │
                    ▼
                 Python
                    │
                    ▼
              server.py
                    │
             uvicorn.run(app)
                    │
                    ▼
                Uvicorn
                    │
                    ▼
              FastAPI app
```

---

# 42. Why Would Someone Use `uvicorn.run(app)`?

For a simple FastAPI project, you don't necessarily need this approach.

The CLI approach is often simpler:

```zsh
fastapi dev server.py
```

or:

```zsh
uvicorn server:app --reload
```

However, starting Uvicorn programmatically can be useful when you want to configure or control the server from Python code.

For example:

```python
uvicorn.run(
    app,
    host="127.0.0.1",
    port=8000,
    reload=True
)
```

This puts the server configuration directly in Python.

It can also be convenient for scripts or certain development setups.

---

# 43. Important Difference When Using Reload

There is an important subtlety when using:

```python
uvicorn.run(app, reload=True)
```

Uvicorn's reload mechanism needs to manage the application process and reload the application when files change.

For reload-related configurations, Uvicorn generally recommends using the import-string form:

```python
uvicorn.run(
    "server:app",
    reload=True
)
```

instead of directly passing:

```python
uvicorn.run(
    app,
    reload=True
)
```

when the application needs to be reloaded.

Therefore, if you are simply learning the basic concept:

```python
uvicorn.run(app)
```

is easy to understand.

For a reload-enabled development setup, the CLI form:

```zsh
uvicorn server:app --reload
```

is often the cleaner approach.

---

# 44. Does `uvicorn.run(app)` Replace FastAPI?

No.

This is another important distinction.

You still have:

```python
app = FastAPI()
```

FastAPI is still the application.

Uvicorn is still the server.

The only difference is **how the server is started**.

```text
FastAPI
   │
   │ creates
   ▼
app object
   │
   │ passed to
   ▼
uvicorn.run(app)
   │
   ▼
Uvicorn
```

---

# 45. Does This Change ASGI?

No.

The ASGI relationship remains the same.

Whether you start the application using:

```zsh
fastapi dev server.py
```

or:

```zsh
uvicorn server:app
```

or:

```python
uvicorn.run(app)
```

the underlying architecture is still:

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
   │
   ▼
Your route function
```

The starting mechanism has changed.

The FastAPI/Uvicorn/ASGI architecture has not.

---

# 46. One Simple Way to Remember All Three

Think about three different people giving Uvicorn instructions.

### FastAPI CLI

You tell FastAPI:

```text
"Here is my file."
```

```zsh
fastapi dev server.py
```

FastAPI CLI takes care of finding the application and starting the server.

---

### Uvicorn CLI

You tell Uvicorn:

```text
"Find the app yourself."
```

```zsh
uvicorn server:app
```

Uvicorn interprets:

```text
server:app
```

as:

```text
server.py → app
```

---

### Python code

You tell Uvicorn:

```text
"Here is the app object."
```

```python
uvicorn.run(app)
```

There is no need for Uvicorn to discover the object through:

```text
server:app
```

because Python has already created the object and passed it directly.

---

# 47. Final Comparison

| Method      | Command/code                            | How app is provided      |
| ----------- | --------------------------------------- | ------------------------ |
| FastAPI CLI | `fastapi dev server.py`                 | FastAPI CLI discovers it |
| Uvicorn CLI | `uvicorn server:app --reload`           | Import string            |
| Python      | `python server.py` + `uvicorn.run(app)` | Actual Python object     |

The core difference is:

```text
fastapi dev server.py
        ↓
FastAPI CLI discovers app

uvicorn server:app
        ↓
Uvicorn imports app

uvicorn.run(app)
        ↓
Python directly gives Uvicorn the app object
```

All three ultimately lead to the same basic architecture:

```text
                    HTTP
                     │
                     ▼
                  Uvicorn
                     │
                    ASGI
                     │
                     ▼
                  FastAPI
                     │
                     ▼
              Your route function
```

So `uvicorn.run(app)` is **not a different kind of FastAPI server**.

It is simply another way of **starting the same Uvicorn server programmatically from Python**.
