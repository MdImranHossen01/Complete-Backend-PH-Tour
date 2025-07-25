# Create a file user.route.ts
```ts
// user/user.route.ts
import { Router } from "express";
import { UserControllers } from "./user.controller";


const router = Router()

router.post("/register", UserControllers.createUser)
router.get("/all-users", UserControllers.getAllUsers)

export const UserRoutes = router

```
---
## update app.ts file

```diff --git a/src/app.ts b/src/app.ts

+import cors from "cors";
 import express, { Request, Response } from "express";
+import { globalErrorHandler } from "./app/middlewares/globalErrorHandler";
+import notFound from "./app/middlewares/notFound";
+import { router } from "./app/routes";
 
 const app = express()
 
+app.use(express.json())
+app.use(cors())
+
+app.use("/api/v1", router)
 
app.get("/", (req: Request, res: Response) => {
    res.status(200).json({
        message: "Welcome to Tour Management System Backend"
    })
})
 
+
+app.use(globalErrorHandler)
+
+app.use(notFound)
+
 export default app
```
---

### Updated app.ts
```ts
import cors from "cors";
import express, { Request, Response } from "express";
import { globalErrorHandler } from "./app/middlewares/globalErrorHandler";
import notFound from "./app/middlewares/notFound";
import { router } from "./app/routes";

const app = express()

app.use(express.json())
app.use(cors())

app.use("/api/v1", router)

app.get("/", (req: Request, res: Response) => {
    res.status(200).json({
        message: "Welcome to Tour Management System Backend"
    })
})


app.use(globalErrorHandler)

app.use(notFound)

export default app
```
---
## create routes folder and index.ts file 
```
📦src
 ┣ 📂app
 ┃ ┣ 📂config
 ┃ ┣ 📂modules
 ┃ ┗ 📂routes
 ┃ ┃ ┗ 📜index.ts
 ┣ 📜app.ts
 ┗ 📜server.ts
 ```
 ### Add the code to index.ts
 ```ts
 import { Router } from "express"
import { UserRoutes } from "../modules/user/user.route"

export const router = Router()

const moduleRoutes = [
    {
        path: "/user",
        route: UserRoutes
    },
    // {
    //     path: "/tour",
    //     route: TourRoutes
    // },
]

moduleRoutes.forEach((route) => {
    router.use(route.path, route.route)
})

// router.use("/user", UserRoutes)
// router.use("/tour", TourRoutes)
// router.use("/division", DivisionRoutes)
// router.use("/booking", BookingRoutes)
// router.use("/user", UserRoutes)
 ```


---
### Create New file user.service.ts

- ***Add the code***
```ts
import { IUser } from "./user.interface";
import { User } from "./user.model";

const createUser = async (payload: Partial<IUser>) => {
    const { name, email } = payload;
    const user = await User.create({
        name,
        email
    })

    return user

}

const getAllUsers = async () => {
    const users = await User.find({});
    const totalUsers = await User.countDocuments();
    return {
        data: users,
        meta: {
            total: totalUsers
        }
    }
};

export const UserServices = {
    createUser,
    getAllUsers
}
```

---
## Create Folder `📂middlewares` and file `globalErrorHandler.ts`, `notFound.ts`
```
📦src
 ┣ 📂app
 ┃ ┣ 📂config
 ┃ ┣ 📂middlewares
 ┃ ┃ ┣ 📜globalErrorHandler.ts
 ┃ ┃ ┗ 📜notFound.ts
 ┃ ┗ 📂routes
 ┣ 📜app.ts
 ┗ 📜server.ts
```
**globalErrorHandler.ts**
```ts
/* eslint-disable @typescript-eslint/no-unused-vars */
/* eslint-disable @typescript-eslint/no-explicit-any */
import { NextFunction, Request, Response } from "express"
import { envVars } from "../config/env"
import AppError from "../errorHelpers/AppError"

export const globalErrorHandler = (err: any, req: Request, res: Response, next: NextFunction) => {

    let statusCode = 500
    let message = "Something Went Wrong!!"

    if (err instanceof AppError) {
        statusCode = err.statusCode
        message = err.message
    } else if (err instanceof Error) {
        statusCode = 500;
        message = err.message
    }

    res.status(statusCode).json({
        success: false,
        message,
        err,
        stack: envVars.NODE_ENV === "development" ? err.stack : null
    })
}
```

**notFound.ts**
```ts
import { Request, Response } from "express";
import httpStatus from "http-status-codes";


const notFound = (req: Request, res: Response) => {
    res.status(httpStatus.NOT_FOUND).json({
        success: false,
        message: "Route Not Found"
    })
}

export default notFound
```

---
## Create Folder `📂errorHelpers` and file `📜AppError.ts`
```
📦src
 ┣ 📂app
 ┃ ┣ 📂config
 ┃ ┣ 📂errorHelpers
 ┃ ┃ ┣ 📜AppError.ts
 ┃ ┗ 📂routes
 ┣ 📜app.ts
 ┗ 📜server.ts
```
**AppError.ts**
```ts
class AppError extends Error {
    public statusCode: number;

    constructor(statusCode: number, message: string, stack = '') {
        super(message) // throw new Error("Something went wrong")
        this.statusCode = statusCode

        if (stack) {
            this.stack = stack
        } else {
            Error.captureStackTrace(this, this.constructor)
        }
    }
}

export default AppError
```
---

## Create Folder `📂utils` and file `📜sendResponse.ts`, `📜catchAsync.ts`
```
📦src
 ┣ 📂app
 ┃ ┣ 📂config
 ┃ ┣ 📂utils
 ┃ ┃ ┣ 📜sendResponse.ts
 ┃ ┃ ┗ 📜catchAsync.ts
 ┃ ┗ 📂routes
 ┣ 📜app.ts
 ┗ 📜server.ts
```
**catchAsync.ts**
```ts
import { NextFunction, Request, Response } from "express";

/* eslint-disable @typescript-eslint/no-explicit-any */
type AsyncHandler = (req: Request, res: Response, next: NextFunction) => Promise<void>

export const catchAsync = (fn: AsyncHandler) => (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch((err: any) => {
        console.log(err);
        next(err)
    })
}
```

**sendResponse.ts**
```ts
import { Response } from "express";

interface TMeta {
    total: number
}


interface TResponse<T> {
    statusCode: number;
    success: boolean;
    message: string;
    data: T;
    meta?: TMeta
}

export const sendResponse = <T>(res: Response, data: TResponse<T>) => {
    res.status(data.statusCode).json({
        statusCode: data.statusCode,
        success: data.success,
        message: data.message,
        meta: data.meta,
        data: data.data
    })
}
```



