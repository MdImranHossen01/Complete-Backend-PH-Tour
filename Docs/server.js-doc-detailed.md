
# 📘 `server.ts` – Detailed Documentation

This file is the **main entry point** of your Node.js + TypeScript application using Express and Mongoose. It connects to MongoDB, starts the HTTP server, and handles graceful shutdowns and unexpected errors.

---

## 📦 1. Imports and Global Variables

```ts
/* eslint-disable no-console */
import { Server } from "http";
import mongoose from "mongoose";
import app from "./app";
import { envVars } from "./app/config/env";

let server: Server;
```

### ✅ Explanation:
- `/* eslint-disable no-console */`: Disables lint warning for using `console.log()`.
- `import { Server } from "http"`: Imports the Server type for strong typing.
- `import mongoose from "mongoose"`: Mongoose is used to connect and interact with MongoDB.
- `import app from "./app"`: Imports your Express app instance.
- `import { envVars } from "./app/config/env"`: Custom config file that loads environment variables like `PORT` and `DB_URL`.
- `let server: Server;`: Declares a variable to hold the HTTP server instance globally.

---

## 🚀 2. Start Server Function

```ts
const startServer = async () => {
    try {
        await mongoose.connect(envVars.DB_URL);
        console.log("Connected to DB!!");

        server = app.listen(envVars.PORT, () => {
            console.log(`Server is listening to port ${envVars.PORT}`);
        });
    } catch (error) {
        console.log(error);
    }
};

startServer();
```

### ✅ Explanation:
- `startServer`: Async function to handle DB connection and server startup.
- `await mongoose.connect(...)`: Connects to MongoDB using URL from `.env`.
- `app.listen(...)`: Starts the Express server and listens on the configured port.
- `server = ...`: Stores the server instance globally for future access.
- `catch (error)`: Catches and logs any error during DB connection or server start.

---

## 🛑 3. Signal Handlers (Graceful Shutdown)

### 🔁 `SIGTERM` Handler

```ts
process.on("SIGTERM", () => {
    console.log("SIGTERM signal received... Server shutting down..");

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});
```

### ⌨️ `SIGINT` Handler

```ts
process.on("SIGINT", () => {
    console.log("SIGINT signal received... Server shutting down..");

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});
```

### ✅ Explanation:
- `SIGTERM`: A termination signal (e.g., Heroku or Docker stop command).
- `SIGINT`: Sent by pressing Ctrl+C in terminal.
- `server.close()`: Ensures server closes all connections before exiting.

---

## ❗ 4. Runtime Error Handling

### 🧵 `unhandledRejection`

```ts
process.on("unhandledRejection", (err) => {
    console.log("Unhandled Rejection detected... Server shutting down..", err);

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});
```

### 🧨 `uncaughtException`

```ts
process.on("uncaughtException", (err) => {
    console.log("Uncaught Exception detected... Server shutting down..", err);

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});
```

### ✅ Explanation:
- `unhandledRejection`: Handles Promise rejections that weren’t caught (no `.catch()`).
- `uncaughtException`: Catches synchronous code errors not wrapped in `try...catch`.
- Both log the error and shutdown the server gracefully.

---

## 🧪 5. Test Examples (Commented)

```ts
// Promise.reject(new Error("I forgot to catch this promise"))
// throw new Error("I forgot to handle this local error")
```

### ✅ Explanation:
- The first simulates a rejected Promise without `.catch()`.
- The second simulates a thrown error in synchronous code.

---

## 📚 References

- [Node.js Process Events](https://nodejs.org/api/process.html)
- [Mongoose Connection](https://mongoosejs.com/docs/connections.html)
- [Express `app.listen()`](https://expressjs.com/en/api.html#app.listen)
- [Graceful Shutdown in Node.js](https://snyk.io/blog/node-js-processes-graceful-shutdown/)

---

## 🧩 Full `server.ts` Code (Reusable)

```ts
/* eslint-disable no-console */
import { Server } from "http";
import mongoose from "mongoose";
import app from "./app";
import { envVars } from "./app/config/env";

let server: Server;

const startServer = async () => {
    try {
        await mongoose.connect(envVars.DB_URL);
        console.log("Connected to DB!!");

        server = app.listen(envVars.PORT, () => {
            console.log(`Server is listening to port ${envVars.PORT}`);
        });
    } catch (error) {
        console.log(error);
    }
};

startServer();

process.on("SIGTERM", () => {
    console.log("SIGTERM signal received... Server shutting down..");

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});

process.on("SIGINT", () => {
    console.log("SIGINT signal received... Server shutting down..");

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});

process.on("unhandledRejection", (err) => {
    console.log("Unhandled Rejection detected... Server shutting down..", err);

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});

process.on("uncaughtException", (err) => {
    console.log("Uncaught Exception detected... Server shutting down..", err);

    if (server) {
        server.close(() => {
            process.exit(1);
        });
    }

    process.exit(1);
});

// Promise.reject(new Error("I forgot to catch this promise"))
// throw new Error("I forgot to handle this local error")
```