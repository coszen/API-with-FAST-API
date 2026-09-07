# FastAPI, Web Servers, Uvicorn and ASGI

## 1. What is a Web Server?

Imagine you have this FastAPI application:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello"}
```

You have written the **application logic**.

But your Python program doesn't automatically know how to:

* Listen for HTTP requests
* Accept connections from browsers
* Understand HTTP
* Send HTTP responses
* Manage multiple connections

That's where a **Web Server** comes in.

Think of the flow as:

```text
Client / Browser
       |
       | HTTP Request
       v
   Web Server
       |
       v
   FastAPI App
       |
       v
 Your Python Function
       |
       v
    Response
       |
       v
     Client
```

For example:

```http
GET /hello
```

comes into the server.

The server passes the request to FastAPI.

FastAPI determines:

```text
/hello → call hello()
```

Your function returns:

```python
{"message": "Hello"}
```

The server then sends the HTTP response back to the client.

---

# 2. What is Uvicorn?

**Uvicorn is an ASGI web server for Python.**

FastAPI is an **ASGI application/framework**.

So:

```text
FastAPI       = Application / Framework
Uvicorn       = Server that runs the application
```

You write:

```python
app = FastAPI()
```

and then run:

```bash
uvicorn main:app
```

Uvicorn loads the `app` object from `main.py` and starts listening for requests.

---

# 3. Why can't FastAPI itself receive HTTP requests?

FastAPI is primarily concerned with **application-level behavior**.

For example:

```python
@app.get("/users")
def get_users():
    return {"users": ["A", "B"]}
```

FastAPI handles things such as:

* Routing
* Request validation
* Response serialization
* Dependency injection
* OpenAPI documentation
* Request/response processing

But something still needs to:

* Listen on a network port
* Accept network connections
* Receive HTTP requests
* Pass requests to the application
* Send responses back

That's the job of the **ASGI server**.

---

# 4. What is ASGI?

This is the key connection.

**ASGI** stands for:

> Asynchronous Server Gateway Interface

It provides a standard interface between:

```text
Web Server <----> Python Web Application
```

So FastAPI doesn't need to know exactly which ASGI server is being used.

Think of ASGI as a **common language/interface**.

```text
                    ASGI
                      |
          +-----------+-----------+
          |                       |
          v                       v
      Uvicorn                 Hypercorn
          |                       |
          v                       v
       FastAPI                 FastAPI
```

Because both servers understand ASGI, they can communicate with FastAPI.

---

# 5. Can FastAPI use another server?

## Yes!

FastAPI does **not** use only Uvicorn.

You can use other ASGI-compatible servers such as:

* Uvicorn
* Hypercorn
* Daphne
* Granian

For example, you can use **Hypercorn** instead of Uvicorn.

Your FastAPI code does not need to change.

### FastAPI application

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello"}
```

### Using Uvicorn

Install:

```bash
pip install uvicorn
```

Run:

```bash
uvicorn main:app
```

### Using Hypercorn

Install:

```bash
pip install hypercorn
```

Run:

```bash
hypercorn main:app
```

The FastAPI code remains the same.

The only thing that changes is the **server running the application**.

---

# 6. Example: Uvicorn + FastAPI

Suppose your project looks like this:

```text
myproject/
│
├── main.py
└── requirements.txt
```

Your `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello World"}
```

Run:

```bash
uvicorn main:app
```

The flow becomes:

```text
Browser
   |
   | GET /hello
   v
Uvicorn
   |
   v
FastAPI
   |
   v
hello()
   |
   v
{"message": "Hello World"}
   |
   v
Browser
```

---

# 7. What does `main:app` mean?

When you run:

```bash
uvicorn main:app
```

the format is:

```text
module:object
```

So:

```text
main:app
```

means:

```text
main → main.py
app  → FastAPI object inside main.py
```

For example:

```python
# main.py

from fastapi import FastAPI

app = FastAPI()
```

Uvicorn finds:

```python
app
```

inside:

```text
main.py
```

and starts serving it.

---

# 8. Why does everyone commonly use Uvicorn with FastAPI?

Because Uvicorn is a very good fit for FastAPI.

It is:

* Lightweight
* High-performance
* ASGI-native
* Designed for asynchronous Python applications
* Supports WebSockets
* Easy to run
* Commonly used with FastAPI

FastAPI's CLI also uses Uvicorn underneath.

For example:

```bash
fastapi dev
```

and:

```bash
fastapi run
```

can use Uvicorn as the underlying server.

---

# 9. Where does Gunicorn fit?

This is where things can become confusing.

You may hear:

> "Use Gunicorn in production."

Gunicorn is a server/process-management solution traditionally associated with **WSGI** applications.

FastAPI is **ASGI**, so you shouldn't think of the architecture simply as:

```text
FastAPI → Gunicorn
```

Instead, you can have:

```text
Gunicorn
    |
    +---- Uvicorn Worker
    |
    +---- Uvicorn Worker
    |
    +---- Uvicorn Worker
    |
    +---- Uvicorn Worker
    |
    v
FastAPI
```

For example:

```bash
gunicorn main:app -w 4 -k uvicorn_worker.UvicornWorker
```

Here:

```text
Gunicorn
   |
   +-- Worker 1 → Uvicorn → FastAPI
   |
   +-- Worker 2 → Uvicorn → FastAPI
   |
   +-- Worker 3 → Uvicorn → FastAPI
   |
   +-- Worker 4 → Uvicorn → FastAPI
```

The benefit is that Gunicorn can manage multiple worker processes while Uvicorn handles the ASGI serving.

---

# 10. Where does Nginx fit?

Now we can look at a common production architecture.

You might have:

```text
                    Internet
                       |
                       v
                    Nginx
                Reverse Proxy
                       |
                       v
                   Uvicorn
                       |
                       v
                   FastAPI
                       |
                       v
                    Database
```

Nginx can act as a **reverse proxy**.

For example, the user accesses:

```text
https://mywebsite.com/users
```

Nginx receives the request and forwards it to your FastAPI server:

```text
Nginx
   |
   v
localhost:8000
   |
   v
Uvicorn
   |
   v
FastAPI
```

This allows Nginx to handle things such as:

* Reverse proxying
* TLS/HTTPS termination
* Static files
* Load balancing
* Connection handling

while FastAPI focuses on application logic.

---

# 11. The complete picture

A production setup could look like:

```text
                    INTERNET
                       |
                       v
              +----------------+
              |     Nginx      |
              | Reverse Proxy  |
              +-------+--------+
                      |
                      v
              +---------------+
              |    Uvicorn    |
              |   ASGI Server |
              +-------+-------+
                      |
                      v
              +---------------+
              |    FastAPI    |
              | Web Framework |
              +-------+-------+
                      |
                      v
              +---------------+
              | Your Python   |
              | Business Logic |
              +-------+-------+
                      |
                      v
                  Database
```

---

# 12. The most important distinction

Keep these concepts separate:

| Component     | What it does                        |
| ------------- | ----------------------------------- |
| **Nginx**     | Reverse proxy / HTTP infrastructure |
| **Uvicorn**   | ASGI web server                     |
| **FastAPI**   | Python web framework                |
| **Your code** | Business/application logic          |
| **Database**  | Stores application data             |

The important relationship is:

```text
FastAPI
   |
   | ASGI interface
   v
Uvicorn
```

Because FastAPI uses the **ASGI standard**, Uvicorn can be replaced by another ASGI-compatible server.

For example:

```text
FastAPI → Uvicorn
```

or:

```text
FastAPI → Hypercorn
```

or:

```text
FastAPI → Daphne
```

or:

```text
FastAPI → Granian
```

---

# 13. Simple mental model

Don't memorize:

> "FastAPI uses Uvicorn."

Instead, remember:

> **FastAPI is an ASGI application. Uvicorn is one ASGI server that can run it.**

The architecture is:

```text
             HTTP Request
                  |
                  v
            +-----------+
            |   Server  |
            |  Uvicorn  |
            +-----+-----+
                  |
                ASGI
                  |
                  v
            +-----------+
            |  FastAPI  |
            +-----+-----+
                  |
                  v
          Your Python Code
                  |
                  v
             HTTP Response
```

That distinction becomes very important when you start learning **FastAPI deployment, Docker, Nginx, Gunicorn, workers, load balancing, and cloud deployment**.
