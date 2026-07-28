#### Python Web Application Framework 
### Flask: The Traditional WSGI Server Framework
### FastAPI: The Modern Async ASGI Server Framework

##### Flask Syntax: Generic @app.route(),
```
# Default GET request
@app.route("/")
def home():
    return "Hello from Flask!"

# Explicitly specifying HTTP methods
@app.route("/submit", methods=["POST", "PUT"])
def submit_data():
```
*Modern Flask added shorthand decorators like @app.get() and @app.post(), but @app.route() remains the standard practice in most Flask codebases*

##### FastAPI Syntax: Method-Specific Decorators
HTTP method in the decorator name itself (@app.get(), @app.post(), @app.delete(), etc.):
```
# Explicit GET route
@app.get("/")
def home():
    return {"message": "Hello from FastAPI!"}

# Explicit POST route
@app.post("/submit")
def submit_data():
    return {"message": "Data submitted!"}
```
#### Decorators 
- the lines starting with `@`, like `@app.get("/stream")` act as the registry or router that exposes your functions to the outside world as `web endpoints`.

#### What the Decorator Does Behind the Scenes
```
@app.get("/stream")
async def stream_logs(request: Request):
    return EventSourceResponse(log_generator(request))
```
###### The @app.get("/stream") decorator tells FastAPI:
- Listen for HTTP requests: "Whenever a user sends an HTTP GET request to the URL path /stream..."
- Execute this function: "...run the stream_logs function directly below me."
- Handle response formatting: Take whatever stream_logs returns (like an HTML response, JSON, or an SSE event stream) and send it back to the client browser over HTTP.

Anatomy of an Endpoint Decorator in FastAPI
```
@app.get("/pingpong")
# ^    ^     ^
# |    |     +--- 3. URL Path
# |    +--------- 2. HTTP Method (get, post, put, delete, patch)
# +-------------- 1. FastAPI Application Instance
```
- `app:` The FastAPI instance you created via app = FastAPI().

- `.get():` Specifies the HTTP verb (method). You can also use .post(), .put(), .delete(), etc.

- `"/pingpong"`: Specifies the URL path path on your server (e.g., http://localhost:5000/pingpong).

##### how a return value from a Python function becomes a web page on a user's browser
```
+-------------------------------------------------------------------------------+
|  1. PYTHON / FASTAPI                                                          |
|  Function returns a string or object:                                         |
|  return "<h1>Hello World</h1>"                                                |
+------------------------------------+------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------------+
|  2. WEB FRAMEWORK & SERVER (FastAPI + Uvicorn)                                |
|  Packs the returned string into a standard HTTP Response packet:              |
|                                                                               |
|  HTTP/1.1 200 OK                                                              |
|  Content-Type: text/html                                                      |
|  Content-Length: 22                                                           |
|                                                                               |
|  <h1>Hello World</h1>                                                         |
+------------------------------------+------------------------------------------+
                                     | (Sent over network)
                                     v
+-------------------------------------------------------------------------------+
|  3. WEB BROWSER (Chrome, Firefox, Safari)                                     |
|  Reads the headers & body, then renders HTML into pixels on screen:           |
|                                                                               |
|  Hello World                                                                  |
+-------------------------------------------------------------------------------+
```

##### Response Headers (Metadata for the browser)
These headers tell the browser what kind of content is coming and how to process it:
- Content-Type: text/html; charset=utf-8 ← Tells the browser: "This is HTML, render it as a visual webpage!"
- HTTP/1.1 200 O
##### Response Body (The actual payload)
This contains the raw data returned by your function:
`<h1>Hello World</h1>`

##### How Response Formatting Varies by Return Type
| Python Return Value | Route Configuration / Response Class | Headers Sent to Browser | How the Browser Displays It |
| :--- | :--- | :--- | :--- |
| `return "<h1>Hi</h1>"` | `response_class=HTMLResponse` | `Content-Type: text/html` | Renders styled HTML elements on the screen. |
| `return {"msg": "pong"}` | Standard (Default) | `Content-Type: application/json` | Displays structured JSON text (useful for APIs). |
| `return EventSourceResponse(...)` | SSE Generator | `Content-Type: text/event-stream` | Keeps connection open to stream live updates. |

--- 
### Flask and fastAPI 
how Flask handles the details automatically differs slightly from FastAPI.
>[!NOTE]
>Flask default text return, FastAPI returns JSON default

The Fundamental Difference: Explicit vs. Implicit
1. Flask (Implicit HTML by default)
In Flask, returning a raw string automatically defaults to treating it as HTML (text/html):

```Python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    # Flask automatically sets header: Content-Type: text/html
    return "<h1>Hello World</h1>"
```
If you want to return JSON in Flask, you usually have to be explicit and use jsonify or return a dictionary (in modern Flask):

```Python
from flask import jsonify

@app.route("/json")
def get_json():
    # Explicitly converts dict to JSON response
    return jsonify({"message": "pong", "counter": 1})
```
2. FastAPI (Explicit HTML, JSON by default)
FastAPI is designed primarily for building JSON APIs. Therefore, returning a raw string or dictionary defaults to treating it as JSON (application/json):

```Python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

# 1. Default behavior -> Sends application/json
@app.get("/json")
def get_json():
    return {"message": "pong", "counter": 1}

# 2. To send HTML, you MUST specify response_class=HTMLResponse
@app.get("/", response_class=HTMLResponse)
def home():
    return "<h1>Hello World</h1>"
```
