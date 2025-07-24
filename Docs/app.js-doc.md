
# 📘 `app.ts` – Express App Setup Documentation

This file initializes the **Express application**. It sets up the basic structure, including JSON parsing and a root route (`GET /`) for API testing or welcome message.

---

## 📦 1. Imports

```ts
import express, { Request, Response } from "express";
```

### ✅ Explanation:
- `express`: Imports the main Express module to create the server app.
- `{ Request, Response }`: TypeScript types for request and response objects, helpful for type-checking.

---

## 🚀 2. Initialize Express App

```ts
const app = express();
```

### ✅ Explanation:
- Creates an instance of the Express application.
- This instance will be used to define routes and middleware.

---

## 🔁 3. Define Root Route

```ts
app.get("/", (req: Request, res: Response) => {
    res.status(200).json({
        message: "Welcome to Tour Management System Backend"
    });
});
```

### ✅ Explanation:
- `app.get("/")`: Defines a GET request handler for the root (`/`) path.
- `(req, res) => { ... }`: Callback function that handles the request and sends a JSON response.
- `res.status(200).json(...)`: Responds with HTTP 200 status and a JSON object with a welcome message.

---

## 🔄 4. Export the App

```ts
export default app;
```

### ✅ Explanation:
- Exports the `app` instance so it can be imported in `server.ts` or other modules.

---

## 🧩 Full `app.ts` Code (Reusable)

```ts
import express, { Request, Response } from "express";

const app = express();

app.get("/", (req: Request, res: Response) => {
    res.status(200).json({
        message: "Welcome to Tour Management System Backend"
    });
});

export default app;
```

---

## 📚 References

- [Express.js Official Docs](https://expressjs.com/)
- [Express `app.get()` Method](https://expressjs.com/en/4x/api.html#app.get)
- [TypeScript with Express](https://expressjs.com/en/advanced/best-practice-performance.html#use-promises)

