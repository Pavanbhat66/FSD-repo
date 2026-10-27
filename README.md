# Week 3: Backend Development with Node.js & Express

A hands-on introduction to building and testing REST-style APIs with Node.js and Express. This week follows a request from the client to the server and back, then applies those ideas in a small in-memory Question Bank API.

## Learning goals

By the end of this material, you should be able to:

- Explain how a Node.js and Express server receives and responds to HTTP requests.
- Choose HTTP methods and resource-oriented endpoint paths.
- Use Postman to inspect requests, responses, and status codes.
- Handle invalid input, missing resources, and repeated delete requests clearly.

## Prerequisites

- Node.js and npm installed
- A terminal
- Postman for sending requests
- Basic JavaScript knowledge

## Quick start

Create a project and install Express:

```bash
mkdir question-bank-api
cd question-bank-api
npm init -y
npm install express
```

Create `server.js` with a minimal server:

```js
const express = require("express");
const app = express();

app.use(express.json());

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

Add a start script to `package.json`:

```json
{
  "scripts": {
    "start": "node server.js"
  }
}
```

Start the server:

```bash
npm start
```

`localhost` refers to your own machine. The port (5000 in this example) identifies the local process receiving the connection. If it is already in use, choose another port or stop the process using it. Keep secrets and environment-specific configuration out of Git by using environment variables.

## Core concepts

### Request-to-response flow

An HTTP request includes a method, URL, headers, and sometimes a body. Express middleware can parse or inspect it before Express matches the method and path to a route handler. The handler runs the application logic and sends a response containing a status code, headers, and usually a body.

```text
Request → middleware → route match → handler → response
```

Use `express.json()` to parse JSON request bodies. For a small exercise, routes and handlers can live in one file. Larger projects often separate route definitions from controller logic.

### REST-style resources

Name the resource in the path and use the HTTP method to express the action. For example, `/api/questions/5` identifies question 5; a query string such as `?tag=fun` is useful for filtering or adjusting a collection request.

| Action | Method and path |
| --- | --- |
| List questions | `GET /api/questions` |
| Get one question | `GET /api/questions/:id` |
| Delete one question | `DELETE /api/questions/:id` |

## Mini Question Bank API

The workshop exercise uses an in-memory array, with no database, authentication, or frontend. Data resets whenever the server restarts.

```js
let questions = [
  { id: 1, question: "Tabs or spaces?" },
  { id: 2, question: "Dark mode or light mode?" }
];
```

Implement the three endpoints listed above. A delete handler should validate the ID, check that the question exists, and only then remove it. One possible behavior is:

```js
app.delete("/api/questions/:id", (req, res) => {
  const id = Number(req.params.id);

  if (!Number.isInteger(id) || id <= 0) {
    return res.status(400).json({ error: "Invalid question ID" });
  }

  const exists = questions.some((question) => question.id === id);
  if (!exists) {
    return res.status(404).json({ error: "Question not found" });
  }

  questions = questions.filter((question) => question.id !== id);
  return res.status(200).json({ message: "Question deleted" });
});
```

Apply the same care to `GET /api/questions/:id`: return the requested question when found and a clear `404` response when it is missing.

## Test with Postman

Start your server, create a request in Postman, and send each request to `http://localhost:5000`:

1. `GET /api/questions` — confirm the list is returned.
2. `GET /api/questions/1` — confirm question 1 is returned.
3. `GET /api/questions/999` — check the missing-question behavior.
4. `DELETE /api/questions/2` — delete an existing question.
5. Repeat `DELETE /api/questions/2` — confirm the missing-resource behavior is handled.
6. `GET /api/unknown` — observe the response for an unknown route.

In Postman, inspect the method and URL, response status, response body, and (when useful) headers. Test both successful requests and edge cases; do not infer success from the message alone.

For practice against a public test API, try `GET https://jsonplaceholder.typicode.com/posts/1` and `GET https://jsonplaceholder.typicode.com/posts`. JSONPlaceholder simulates write requests; changes do not persist as real data.

## Status codes used in the exercise

| Code | Meaning in this API |
| --- | --- |
| `200 OK` | The request succeeded, such as listing a question or completing a deletion. |
| `400 Bad Request` | The supplied question ID is invalid. |
| `404 Not Found` | The question or route does not exist. |
| `500 Internal Server Error` | An unexpected server-side failure occurred. |

Choose a status that matches what happened, and return a useful error message when a request cannot be fulfilled.

## Troubleshooting

- **`Cannot GET ...`** — check that the route exists and the method and URL are correct.
- **`EADDRINUSE`** — the chosen port is already occupied.
- **`Cannot find module 'express'`** — install dependencies from the project directory with `npm install express`.
- **`ECONNREFUSED 127.0.0.1:5000`** — make sure the server is running and listening on that port.
- **CORS error in a browser** — inspect the server's cross-origin configuration. Postman is useful for separating browser CORS behavior from whether an endpoint responds.

## Included Week 3 resources

- `week3_backend_node_express_workshop.pptx` — workshop slides covering Node.js, Express, request/response flow, REST design, Postman, the mini API, edge cases, and debugging.
- `cheatsheet.pdf` — reference sheet supplied with the workshop.
- `mini_question_bank_three_endpoint_solutions.pdf` — solutions resource for the three-endpoint Question Bank exercise.
- `text.txt` — contains the JSONPlaceholder practice URL: `https://jsonplaceholder.typicode.com/posts/1`.

## Suggested workflow

Design the resource paths, implement the happy path, test it in Postman, then test invalid IDs, missing questions, repeated deletes, and unknown routes. Improve the behavior based on what the response actually tells the client.
