# Setting Up Server and App
- [Module 25](https://web.programming-hero.com/level2-batch-5/video/level2-batch-5-25-9-setting-up-server-and-app)
- add `"dev" : "ts-node-dev --respawn --transpile-only ./src/server.ts",` to `package.json`

- ***server.ts*** 
```javascript
// This line disables a specific ESLint rule (`no-console`) for this file.
// It prevents ESLint from showing a warning when we use `console.log`.
// While `console.log` is often removed in production code, it's kept here for debugging purposes.
/* eslint-disable no-console */

// We are importing the `Server` type from Node.js's built-in `http` module.
// This helps with TypeScript type checking for our server variable.
import { Server } from "http";

// Importing the Mongoose library, which helps us interact with our MongoDB database easily.
import mongoose from "mongoose";

// Importing our custom Express application from the `app.ts` file.
// This file contains all our routes, middleware, etc.
import app from "./app";

// Importing our environment variables (loaded from the `.env` file).
// This includes sensitive data like the port number and database URL.
import { envVars } from "./app/config/env";

// Declaring a variable that will hold our server instance after it starts.
// We'll use this later to close the server gracefully.
let server: Server;

// Defining an asynchronous function that will handle the entire process of starting our server and connecting to the database.
const startServer = async () => {
  // Using a `try...catch` block to handle potential errors during startup.
  // This prevents the app from crashing if the database connection fails or the server can't start.
  try {
    // Connecting to the MongoDB database using Mongoose.
    // The `await` keyword ensures that we wait for the connection to be established before moving to the next line.
    await mongoose.connect(envVars.DB_URL);

    // Logging a success message to the console once the database connection is successful.
    console.log("Connected to DB!!");

    // Starting our Express app on the specified port.
    // `app.listen` returns a server object, which we store in our `server` variable.
    server = app.listen(envVars.PORT, () => {
      // Logging a message to indicate which port the server is running on.
      console.log(`Server is listening to port ${envVars.PORT}`);
    });
  } catch (error) {
    // If any error occurs in the `try` block, it will be caught here and logged to the console.
    console.log(error);
  }
};

// Calling the function to start the server.
startServer();

// `SIGTERM` is a signal sent to a process to request its termination (a graceful shutdown).
// This often comes from hosting platforms. This code listens for that signal.
process.on("SIGTERM", () => {
  console.log("SIGTERM signal received... Server shutting down..");
  // Check if the server is running.
  if (server) {
    // `server.close()` stops accepting new connections and waits for existing ones to finish before shutting down.
    server.close(() => {
      // Exit the process after the server has closed. An exit code of 1 indicates termination with an error.
      process.exit(1);
    });
  } else {
    // If the server isn't running, exit immediately.
    process.exit(1);
  }
});

// `SIGINT` is the signal sent when you press `Ctrl+C` in the terminal. This also triggers a graceful shutdown.
process.on("SIGINT", () => {
  console.log("SIGINT signal received... Server shutting down..");
  if (server) {
    server.close(() => {
      process.exit(1);
    });
  } else {
    process.exit(1);
  }
});


// This event listener catches any promise rejections that were not handled with a `.catch()` block.
// It's a global safety net for asynchronous errors.
process.on("unhandledRejection", (err) => {
  console.log("Unhandled Rejection detected... Server shutting down..", err);

  // If the server is running, close it gracefully to avoid leaving requests hanging.
  if (server) {
    server.close(() => {
      process.exit(1);
    });
  } else {
    // If the server isn't running, exit immediately.
    process.exit(1);
  }
});

// This event listener catches any synchronous errors that were not caught with a `try...catch` block.
// This is the last line of defense to prevent the entire application from crashing.
process.on("uncaughtException", (err) => {
  console.log("Uncaught Exception detected... Server shutting down..", err);

  if (server) {
    server.close(() => {
      process.exit(1);
    });
  } else {
    process.exit(1);
  }
});


// The lines below are examples of how you could trigger the errors handled above.

// Example of how to trigger an "unhandledRejection" error:
// Promise.reject(new Error("I forgot to catch this promise"))

// Example of how to trigger an "uncaughtException" error:
// throw new Error("I forgot to handle this local error")
```
---
***app.ts***
```javascript
// Importing the express library and specific types from it
import express, { Request, Response } from "express"; 

// Initialize the express application
const app = express();

/**
 * Route: GET /
 * Description: This is the root route of your API. 
 * When someone accesses the root URL ("/"), it will return a JSON message.
 * 
 * req: The incoming request object (contains request info)
 * res: The outgoing response object (used to send a response)
 */
app.get("/", (req: Request, res: Response) => {
  // Sending a 200 OK status with a welcome message in JSON format
  res.status(200).json({
    message: "Welcome to Tour Management System Backend",
  });
});

// Exporting the app object so it can be used in another file like server.ts or index.ts
export default app;

```
---
