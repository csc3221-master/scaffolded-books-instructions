# Phase 1 – server.js Explained

This document explains the structure and purpose of each logical block in the Phase-1 `server.js` file. The goal is not just what the code does, but why each part exists.

## 1. Imports – required Node modules
```js
const http = require("http");
const fs = require("fs");
const path = require("path");
const { URLSearchParams } = require("url");
```
Purpose:
- `http`: create a raw HTTP server (no Express yet)
- `fs`: read files from disk (index.html)
- `path`: safely build file paths across operating systems
- `URLSearchParams`: parse form-encoded POST bodies

## 2. Configuration constants
```js
const PORT = process.env.PORT || 3000;
```

Why this matters:
- Defaults to port 3000 for local development
- Allows overrides via environment variables (Docker, cloud, CI)

## 3. Helper functions

### 3.1 Sending a response
```js
function send(res, statusCode, body, contentType = "text/html; charset=utf-8") {
  res.writeHead(statusCode, { "Content-Type": contentType });
  res.end(body);
}
```
Purpose:
- Avoids repeating `writeHead` + `end`
- Makes intent explicit: send a response


### 3.2 Reading a file
```js
function readFileSafe(filePath) {
  return fs.readFileSync(filePath, "utf8");
}
```

Purpose:
- Keeps routing logic readable
- Isolates file-system concerns
- Makes later refactoring easier (async, caching, etc.)


## 4. Creating the HTTP server

const server = http.createServer((req, res) => {

This callback runs once per HTTP request.

Conceptually:

Every request flows through this function.

⸻

5. Normalize request information

  const method = req.method || "GET";
  const url = req.url || "/";

Why:
	•	Defensive coding
	•	Makes routing conditions easy to read and reason about

⸻

6. Route: GET / – serve the HTML form

  if (method === "GET" && url === "/") {
    const filePath = path.join(__dirname, "public", "index.html");
    try {
      const html = readFileSafe(filePath);
      return send(res, 200, html);
    } catch (err) {
      return send(res, 500, "<h1>500</h1><p>Could not load index.html</p>");
    }
  }

What happens here:
	1.	Match the route (GET /)
	2.	Build a safe path to index.html
	3.	Read the file
	4.	Send it as HTML

Why try/catch:
	•	Prevents server crashes
	•	Returns a meaningful error if the file is missing

⸻

7. Route: POST /submit – handle form submission

  if (method === "POST" && url === "/submit") {

This block handles data submitted by the HTML form.

⸻

7.1 Collecting the request body

    let body = "";

    req.on("data", (chunk) => {
      body += chunk.toString("utf8");
      if (body.length > 1_000_000) req.destroy();
    });

Key ideas:
	•	HTTP request bodies arrive in chunks
	•	We accumulate them manually
	•	A size limit prevents abuse

This teaches that HTTP is stream-based, not function-based.

⸻

7.2 Processing once the body is complete

    req.on("end", () => {

This event fires when all data has arrived.

⸻

7.3 Parsing form data

      const params = new URLSearchParams(body);
      const name = (params.get("name") || "").trim();

HTML forms submit data in this format:

name=Carlos

URLSearchParams converts it into usable key-value pairs.

⸻

7.4 Escaping user input

      const safeName = name
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;");

Why this matters:
	•	Prevents HTML injection
	•	Models basic security hygiene early

⸻

7.5 Sending the response HTML

      return send(res, 200, `<!doctype html> ...`);

This shows:
	•	Servers can generate HTML dynamically
	•	Data flows browser → server → browser

⸻

8. Fallback route – 404

  send(res, 404, "<h1>404</h1><p>Not found</p>");

Purpose:
	•	Guarantees every request gets a response
	•	Avoids hanging connections
	•	Makes behavior explicit

⸻

9. Start listening for requests

server.listen(PORT, "0.0.0.0", () => {
  console.log(`Server running at http://localhost:${PORT}`);
});

Why 0.0.0.0:
	•	Required for Docker port forwarding
	•	Allows access from the host machine

⸻

Big-picture takeaway

Phase 1 intentionally shows:
	•	What an HTTP server really is
	•	How routing works without frameworks
	•	How form data is transmitted and parsed

This makes Phase 2 (Express) feel earned rather than magical.