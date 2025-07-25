# 🧩 User Controller File Explanation (Beginner Friendly)

This file defines controller logic for handling user-related routes in an Express.js + TypeScript project.

---

### instal:
```
npm i http-status-codes
```

## ✅ Full Code

```ts
/* eslint-disable @typescript-eslint/no-unused-vars */

import { NextFunction, Request, Response } from "express";
import httpStatus from "http-status-codes";
import { catchAsync } from "../../utils/catchAsync";
import { sendResponse } from "../../utils/sendResponse";
import { UserServices } from "./user.service";

// const createUserFunction = async (req: Response, res: Response) => {
//     const user = await UserServices.createUser(req.body)
//     res.status(httpStatus.CREATED).json({
//         message: "User Created Successfully",
//         user
//     })
// }

// const createUser = async (req: Request, res: Response, next: NextFunction) => {
//     try {
//         // throw new Error("Fake error")
//         // throw new AppError(httpStatus.BAD_REQUEST, "fake error")
//         // createUserFunction(req, res)
//     } catch (err: any) {
//         console.log(err);
//         next(err)
//     }
// }

const createUser = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const user = await UserServices.createUser(req.body)

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.CREATED,
        message: "User Created Successfully",
        data: user,
    })
})

const getAllUsers = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const result = await UserServices.getAllUsers();

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.CREATED,
        message: "All Users Retrieved Successfully",
        data: result.data,
        meta: result.meta
    })
})

export const UserControllers = {
    createUser,
    getAllUsers
}

// route matching -> controller -> service -> model -> DB
```

---

## 📘 Line-by-Line Explanation

### 🛡️ ESLint Disable Comment

```ts
/* eslint-disable @typescript-eslint/no-unused-vars */
```
Temporarily disables the ESLint rule that warns about unused variables (like `next`).

---

### 📥 Imports

- `Request`, `Response`, `NextFunction`: Express types for HTTP request, response, and middleware functions.
- `httpStatus`: Standard HTTP status codes (like 201, 400, etc.).
- `catchAsync`: A wrapper function to catch errors in async functions.
- `sendResponse`: A custom utility function to send structured JSON responses.
- `UserServices`: Business logic layer for creating and retrieving users.

---

### ⚙️ `createUser` Controller

```ts
const createUser = catchAsync(async (req, res, next) => {...})
```

- Uses `catchAsync` to handle errors automatically.
- Extracts user data from `req.body` and passes it to the service layer.
- Uses `sendResponse()` to format and send a success response with:
  - `statusCode`: 201 Created
  - `message`: Confirmation message
  - `data`: The newly created user object

---

### 👥 `getAllUsers` Controller

```ts
const getAllUsers = catchAsync(async (req, res, next) => {...})
```

- Fetches all users using the service layer.
- Sends a structured response with:
  - `data`: List of users
  - `meta`: Pagination or meta info (if any)

---

### 🚀 Exporting All Controllers

```ts
export const UserControllers = {
    createUser,
    getAllUsers
}
```

Exports the functions as an object, so they can be imported and used in route definitions.

---

### 📌 Architecture Note

```ts
// route matching -> controller -> service -> model -> DB
```

Describes the typical request flow:
1. Route is matched.
2. Controller is called.
3. Controller calls a service.
4. Service interacts with model.
5. Model communicates with the database.

---

## 📝 Summary

- This controller handles user creation and retrieval.
- Uses helper functions like `catchAsync` and `sendResponse` for clean code.
- Separates concerns between controller and service layers.
- Ideal for scalable and maintainable applications.