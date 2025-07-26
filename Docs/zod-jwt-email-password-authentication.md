# Step-1: Zod Validation
---
### Update `user.route.ts`
---
```diff

 import { Router } from "express";
+import { checkAuth } from "../../middlewares/checkAuth";
+import { validateRequest } from "../../middlewares/validateRequest";
 import { UserControllers } from "./user.controller";
-
+import { Role } from "./user.interface";
+import { createUserZodSchema, updateUserZodSchema } from "./user.validation";
 
 const router = Router()
 
-router.post("/register", UserControllers.createUser)
-router.get("/all-users", UserControllers.getAllUsers)
 
+
+router.post("/register", validateRequest(createUserZodSchema), UserControllers.createUser)
+router.get("/all-users", checkAuth(Role.ADMIN, Role.SUPER_ADMIN), UserControllers.getAllUsers)
+router.patch("/:id", validateRequest(updateUserZodSchema), checkAuth(...Object.values(Role)), UserControllers.updateUser)
+// /api/v1/user/:id
 export const UserRoutes = router
```
---
### Updated code
---
```ts
// src/app/modules/user/user.route.ts
import { Router } from "express";
import { checkAuth } from "../../middlewares/checkAuth";
import { validateRequest } from "../../middlewares/validateRequest";
import { UserControllers } from "./user.controller";
import { Role } from "./user.interface";
import { createUserZodSchema, updateUserZodSchema } from "./user.validation";

const router = Router()



router.post("/register", validateRequest(createUserZodSchema), UserControllers.createUser)
router.get("/all-users", checkAuth(Role.ADMIN, Role.SUPER_ADMIN), UserControllers.getAllUsers)
router.patch("/:id", validateRequest(updateUserZodSchema), checkAuth(...Object.values(Role)), UserControllers.updateUser)
// /api/v1/user/:id
export const UserRoutes = router
```


### Create new File `user.validation.ts` at `src/app/modules/user/user.validation.ts`

```ts
import z from "zod";
import { IsActive, Role } from "./user.interface";

export const createUserZodSchema = z.object({
    name: z
        .string({ invalid_type_error: "Name must be string" })
        .min(2, { message: "Name must be at least 2 characters long." })
        .max(50, { message: "Name cannot exceed 50 characters." }),
    email: z
        .string({ invalid_type_error: "Email must be string" })
        .email({ message: "Invalid email address format." })
        .min(5, { message: "Email must be at least 5 characters long." })
        .max(100, { message: "Email cannot exceed 100 characters." }),
    password: z
        .string({ invalid_type_error: "Password must be string" })
        .min(8, { message: "Password must be at least 8 characters long." })
        .regex(/^(?=.*[A-Z])/, {
            message: "Password must contain at least 1 uppercase letter.",
        })
        .regex(/^(?=.*[!@#$%^&*])/, {
            message: "Password must contain at least 1 special character.",
        })
        .regex(/^(?=.*\d)/, {
            message: "Password must contain at least 1 number.",
        }),
    phone: z
        .string({ invalid_type_error: "Phone Number must be string" })
        .regex(/^(?:\+8801\d{9}|01\d{9})$/, {
            message: "Phone number must be valid for Bangladesh. Format: +8801XXXXXXXXX or 01XXXXXXXXX",
        })
        .optional(),
    address: z
        .string({ invalid_type_error: "Address must be string" })
        .max(200, { message: "Address cannot exceed 200 characters." })
        .optional()
})
export const updateUserZodSchema = z.object({
    name: z
        .string({ invalid_type_error: "Name must be string" })
        .min(2, { message: "Name must be at least 2 characters long." })
        .max(50, { message: "Name cannot exceed 50 characters." }).optional(),
    password: z
        .string({ invalid_type_error: "Password must be string" })
        .min(8, { message: "Password must be at least 8 characters long." })
        .regex(/^(?=.*[A-Z])/, {
            message: "Password must contain at least 1 uppercase letter.",
        })
        .regex(/^(?=.*[!@#$%^&*])/, {
            message: "Password must contain at least 1 special character.",
        })
        .regex(/^(?=.*\d)/, {
            message: "Password must contain at least 1 number.",
        }).optional(),
    phone: z
        .string({ invalid_type_error: "Phone Number must be string" })
        .regex(/^(?:\+8801\d{9}|01\d{9})$/, {
            message: "Phone number must be valid for Bangladesh. Format: +8801XXXXXXXXX or 01XXXXXXXXX",
        })
        .optional(),
    role: z
        // .enum(["ADMIN", "GUIDE", "USER", "SUPER_ADMIN"])
        .enum(Object.values(Role) as [string])
        .optional(),
    isActive: z
        .enum(Object.values(IsActive) as [string])
        .optional(),
    isDeleted: z
        .boolean({ invalid_type_error: "isDeleted must be true or false" })
        .optional(),
    isVerified: z
        .boolean({ invalid_type_error: "isVerified must be true or false" })
        .optional(),
    address: z
        .string({ invalid_type_error: "Address must be string" })
        .max(200, { message: "Address cannot exceed 200 characters." })
        .optional()
})
```


### Create new File `validateRequest.ts` at `src/app/middlewares/validateRequest.ts`

```ts
import { NextFunction, Request, Response } from "express"
import { AnyZodObject } from "zod"

export const validateRequest = (zodSchema: AnyZodObject) => async (req: Request, res: Response, next: NextFunction) => {
    try {
        req.body = await zodSchema.parseAsync(req.body)
        next()
    } catch (error) {
        next(error)
    }
}
```


### Update `user.service.ts`


```diff
-import { IUser } from "./user.interface";
+import bcryptjs from "bcryptjs";
+import httpStatus from "http-status-codes";
+import { JwtPayload } from "jsonwebtoken";
+import { envVars } from "../../config/env";
+import AppError from "../../errorHelpers/AppError";
+import { IAuthProvider, IUser, Role } from "./user.interface";
 import { User } from "./user.model";
 
 const createUser = async (payload: Partial<IUser>) => {
-    const { name, email } = payload;
+    const { email, password, ...rest } = payload;
+
+    const isUserExist = await User.findOne({ email })
+
+    if (isUserExist) {
+        throw new AppError(httpStatus.BAD_REQUEST, "User Already Exist")
+    }
+
+    const hashedPassword = await bcryptjs.hash(password as string, Number(envVars.BCRYPT_SALT_ROUND))
+
+    const authProvider: IAuthProvider = { provider: "credentials", providerId: email as string }
+
+
     const user = await User.create({
-        name,
-        email
+        email,
+        password: hashedPassword,
+        auths: [authProvider],
+        ...rest
     })
 
     return user
 
 }
 
+const updateUser = async (userId: string, payload: Partial<IUser>, decodedToken: JwtPayload) => {
+
+    const ifUserExist = await User.findById(userId);
+
+    if (!ifUserExist) {
+        throw new AppError(httpStatus.NOT_FOUND, "User Not Found")
+    }
+
+    /**
+     * email - can not update
+     * name, phone, password address
+     * password - re hashing
+     *  only admin superadmin - role, isDeleted...
+     * 
+     * promoting to superadmin - superadmin
+     */
+
+    if (payload.role) {
+        if (decodedToken.role === Role.USER || decodedToken.role === Role.GUIDE) {
+            throw new AppError(httpStatus.FORBIDDEN, "You are not authorized");
+        }
+
+        if (payload.role === Role.SUPER_ADMIN && decodedToken.role === Role.ADMIN) {
+            throw new AppError(httpStatus.FORBIDDEN, "You are not authorized");
+        }
+    }
+
+    if (payload.isActive || payload.isDeleted || payload.isVerified) {
+        if (decodedToken.role === Role.USER || decodedToken.role === Role.GUIDE) {
+            throw new AppError(httpStatus.FORBIDDEN, "You are not authorized");
+        }
+    }
+
+    if (payload.password) {
+        payload.password = await bcryptjs.hash(payload.password, envVars.BCRYPT_SALT_ROUND)
+    }
+
+    const newUpdatedUser = await User.findByIdAndUpdate(userId, payload, { new: true, runValidators: true })
+
+    return newUpdatedUser
+}
+
+
 const getAllUsers = async () => {
     const users = await User.find({});
     const totalUsers = await User.countDocuments();

 
 export const UserServices = {
     createUser,
-    getAllUsers
+    getAllUsers,
+    updateUser
 }
```

### Updated code 

```ts
import bcryptjs from "bcryptjs";
import httpStatus from "http-status-codes";
import { JwtPayload } from "jsonwebtoken";
import { envVars } from "../../config/env";
import AppError from "../../errorHelpers/AppError";
import { IAuthProvider, IUser, Role } from "./user.interface";
import { User } from "./user.model";

const createUser = async (payload: Partial<IUser>) => {
    const { email, password, ...rest } = payload;

    const isUserExist = await User.findOne({ email })

    if (isUserExist) {
        throw new AppError(httpStatus.BAD_REQUEST, "User Already Exist")
    }

    const hashedPassword = await bcryptjs.hash(password as string, Number(envVars.BCRYPT_SALT_ROUND))

    const authProvider: IAuthProvider = { provider: "credentials", providerId: email as string }


    const user = await User.create({
        email,
        password: hashedPassword,
        auths: [authProvider],
        ...rest
    })

    return user

}

const updateUser = async (userId: string, payload: Partial<IUser>, decodedToken: JwtPayload) => {

    const ifUserExist = await User.findById(userId);

    if (!ifUserExist) {
        throw new AppError(httpStatus.NOT_FOUND, "User Not Found")
    }

    /**
     * email - can not update
     * name, phone, password address
     * password - re hashing
     *  only admin superadmin - role, isDeleted...
     * 
     * promoting to superadmin - superadmin
     */

    if (payload.role) {
        if (decodedToken.role === Role.USER || decodedToken.role === Role.GUIDE) {
            throw new AppError(httpStatus.FORBIDDEN, "You are not authorized");
        }

        if (payload.role === Role.SUPER_ADMIN && decodedToken.role === Role.ADMIN) {
            throw new AppError(httpStatus.FORBIDDEN, "You are not authorized");
        }
    }

    if (payload.isActive || payload.isDeleted || payload.isVerified) {
        if (decodedToken.role === Role.USER || decodedToken.role === Role.GUIDE) {
            throw new AppError(httpStatus.FORBIDDEN, "You are not authorized");
        }
    }

    if (payload.password) {
        payload.password = await bcryptjs.hash(payload.password, envVars.BCRYPT_SALT_ROUND)
    }

    const newUpdatedUser = await User.findByIdAndUpdate(userId, payload, { new: true, runValidators: true })

    return newUpdatedUser
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
    getAllUsers,
    updateUser
}
```

### Update `user.interface.ts`

```ts
import { Types } from "mongoose";

export enum Role {
    SUPER_ADMIN = "SUPER_ADMIN",
    ADMIN = "ADMIN",
    USER = "USER",
    GUIDE = "GUIDE",
}

//auth providers
/**
 * email, password 
 * google authentication
 */

export interface IAuthProvider {
    provider: "google" | "credentials";  // "Google", "Credential"
    providerId: string;
}

export enum IsActive {
    ACTIVE = "ACTIVE",
    INACTIVE = "INACTIVE",
    BLOCKED = "BLOCKED"
}

export interface IUser {
    name: string;
    email: string;
    password?: string;
    phone?: string;
    picture?: string;
    address?: string;
    isDeleted?: string;
    isActive?: IsActive;
    isVerified?: boolean;
    role: Role;
    auths: IAuthProvider[]
    bookings?: Types.ObjectId[]
    guides?: Types.ObjectId[]
}
```

# Step-2 Password Hashing

- install
```
npm i bcryptjs
```
- install
```
npm i -D @types/bcryptjs
```

### Create new folder and file `auth`
```
📦modules
 ┣ 📂auth
 ┃ ┣ 📜auth.controller.ts
 ┃ ┣ 📜auth.route.ts
 ┃ ┗ 📜auth.service.ts

``` 
### 📜auth.service.ts
```ts
import bcryptjs from "bcryptjs";
import httpStatus from "http-status-codes";
import { envVars } from "../../config/env";
import AppError from "../../errorHelpers/AppError";
import { generateToken } from "../../utils/jwt";
import { IUser } from "../user/user.interface";
import { User } from "../user/user.model";


const credentialsLogin = async (payload: Partial<IUser>) => {
    const { email, password } = payload;

    const isUserExist = await User.findOne({ email })

    if (!isUserExist) {
        throw new AppError(httpStatus.BAD_REQUEST, "Email does not exist")
    }

    const isPasswordMatched = await bcryptjs.compare(password as string, isUserExist.password as string)

    if (!isPasswordMatched) {
        throw new AppError(httpStatus.BAD_REQUEST, "Incorrect Password")
    }
    const jwtPayload = {
        userId: isUserExist._id,
        email: isUserExist.email,
        role: isUserExist.role
    }
    const accessToken = generateToken(jwtPayload, envVars.JWT_ACCESS_SECRET, envVars.JWT_ACCESS_EXPIRES)

    return {
        accessToken
    }

}

//user - login - token (email, role, _id) - booking / payment / booking / payment cancel - token 

export const AuthServices = {
    credentialsLogin
}
```
### 📜auth.controller.ts
```ts
/* eslint-disable @typescript-eslint/no-unused-vars */
import { NextFunction, Request, Response } from "express"
import httpStatus from "http-status-codes"
import { catchAsync } from "../../utils/catchAsync"
import { sendResponse } from "../../utils/sendResponse"
import { AuthServices } from "./auth.service"

const credentialsLogin = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const loginInfo = await AuthServices.credentialsLogin(req.body)

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.OK,
        message: "User Logged In Successfully",
        data: loginInfo,
    })
})

export const AuthControllers = {
    credentialsLogin
}
```
### 📜auth.route.ts
```ts
import { Router } from "express";
import { AuthControllers } from "./auth.controller";

const router = Router()

router.post("/login", AuthControllers.credentialsLogin)

export const AuthRoutes = router;
```

### update  routes/index.ts
```ts
import { Router } from "express"
import { AuthRoutes } from "../modules/auth/auth.route"
import { UserRoutes } from "../modules/user/user.route"

export const router = Router()

const moduleRoutes = [
    {
        path: "/user",
        route: UserRoutes
    },
    {
        path: "/auth",
        route: AuthRoutes
    }
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

# Step-3 JWT Implement

### Install jsonwebtoken
```
npm i jsonwebtoken
npm i -D @types/jsonwebtoken
```
### Update `user.route.ts` at `src/app/modules/user/user.route.ts`

```ts
import { Router } from "express";
import { checkAuth } from "../../middlewares/checkAuth";
import { validateRequest } from "../../middlewares/validateRequest";
import { UserControllers } from "./user.controller";
import { Role } from "./user.interface";
import { createUserZodSchema, updateUserZodSchema } from "./user.validation";

const router = Router()



router.post("/register", validateRequest(createUserZodSchema), UserControllers.createUser)
router.get("/all-users", checkAuth(Role.ADMIN, Role.SUPER_ADMIN), UserControllers.getAllUsers)
router.patch("/:id", validateRequest(updateUserZodSchema), checkAuth(...Object.values(Role)), UserControllers.updateUser)
// /api/v1/user/:id
export const UserRoutes = router
```


### Create `jwt.ts` at `src/app/utils/jwt.ts`
```ts
import jwt, { JwtPayload, SignOptions } from "jsonwebtoken"

export const generateToken = (payload: JwtPayload, secret: string, expiresIn: string) => {
    const token = jwt.sign(payload, secret, {
        expiresIn
    } as SignOptions)

    return token
}

export const verifyToken = (token: string, secret: string) => {

    const verifiedToken = jwt.verify(token, secret);

    return verifiedToken
}
```

### Update `.env`
```ts
PORT=5000
DB_URL=mongodb+srv://example_db_user_name:example_db_password@cluster0.edawjnd.mongodb.net/example-db?retryWrites=true&w=majority&appName=Cluster0
NODE_ENV=development

# JWT
JWT_ACCESS_SECRET=access_secret
JWT_ACCESS_EXPIRES=1d

# BCRYPT
BCRYPT_SALT_ROUND=10

# SUPER ADMIN
SUPER_ADMIN_EMAIL=super@gmail.com
SUPER_ADMIN_PASSWORD=12345678
```
### Update `env.ts` at `src/app/config/env.ts`
```ts
import dotenv from "dotenv";

dotenv.config()

interface EnvConfig {
    PORT: string,
    DB_URL: string,
    NODE_ENV: "development" | "production"
    BCRYPT_SALT_ROUND: string
    JWT_ACCESS_EXPIRES: string
    JWT_ACCESS_SECRET: string
    SUPER_ADMIN_EMAIL: string
    SUPER_ADMIN_PASSWORD: string
}

const loadEnvVariables = (): EnvConfig => {
    const requiredEnvVariables: string[] = ["PORT", "DB_URL", "NODE_ENV", "BCRYPT_SALT_ROUND", "JWT_ACCESS_EXPIRES", "JWT_ACCESS_SECRET", "SUPER_ADMIN_EMAIL", "SUPER_ADMIN_PASSWORD"];

    requiredEnvVariables.forEach(key => {
        if (!process.env[key]) {
            throw new Error(`Missing require environment variabl ${key}`)
        }
    })

    return {
        PORT: process.env.PORT as string,
        // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
        DB_URL: process.env.DB_URL!,
        NODE_ENV: process.env.NODE_ENV as "development" | "production",
        BCRYPT_SALT_ROUND: process.env.BCRYPT_SALT_ROUND as string,
        JWT_ACCESS_SECRET: process.env.JWT_ACCESS_SECRET as string,
        JWT_ACCESS_EXPIRES: process.env.JWT_ACCESS_EXPIRES as string,
        SUPER_ADMIN_EMAIL: process.env.SUPER_ADMIN_EMAIL as string,
        SUPER_ADMIN_PASSWORD: process.env.SUPER_ADMIN_PASSWORD as string
    }
}

export const envVars = loadEnvVariables()
```

### Create `checkAuth.ts` at middleware `src/app/middlewares/checkAuth.ts`
```ts
import { NextFunction, Request, Response } from "express";
import { JwtPayload } from "jsonwebtoken";
import { envVars } from "../config/env";
import AppError from "../errorHelpers/AppError";
import { verifyToken } from "../utils/jwt";

export const checkAuth = (...authRoles: string[]) => async (req: Request, res: Response, next: NextFunction) => {

    try {
        const accessToken = req.headers.authorization;

        if (!accessToken) {
            throw new AppError(403, "No Token Recieved")
        }


        const verifiedToken = verifyToken(accessToken, envVars.JWT_ACCESS_SECRET) as JwtPayload
        if (!authRoles.includes(verifiedToken.role)) {
            throw new AppError(403, "You are not permitted to view this route!!!")
        }
        req.user = verifiedToken
        next()

    } catch (error) {
        console.log("jwt error", error);
        next(error)
    }
}
```

## Super admin work
---
### Create `seedSuperAdmin.ts` at utils `src/app/utils/seedSuperAdmin.ts`

```ts
import bcryptjs from "bcryptjs";
import { envVars } from "../config/env";
import { IAuthProvider, IUser, Role } from "../modules/user/user.interface";
import { User } from "../modules/user/user.model";

export const seedSuperAdmin = async () => {
    try {
        const isSuperAdminExist = await User.findOne({ email: envVars.SUPER_ADMIN_EMAIL })

        if (isSuperAdminExist) {
            console.log("Super Admin Already Exists!");
            return;
        }

        console.log("Trying to create Super Admin...");

        const hashedPassword = await bcryptjs.hash(envVars.SUPER_ADMIN_PASSWORD, Number(envVars.BCRYPT_SALT_ROUND))

        const authProvider: IAuthProvider = {
            provider: "credentials",
            providerId: envVars.SUPER_ADMIN_EMAIL
        }

        const payload: IUser = {
            name: "Super admin",
            role: Role.SUPER_ADMIN,
            email: envVars.SUPER_ADMIN_EMAIL,
            password: hashedPassword,
            isVerified: true,
            auths: [authProvider]

        }

        const superadmin = await User.create(payload)
        console.log("Super Admin Created Successfuly! \n");
        console.log(superadmin);
    } catch (error) {
        console.log(error);
    }
}
```
### Update `server.ts`
```ts
/* eslint-disable no-console */
import { Server } from "http";
import mongoose from "mongoose";
import app from "./app";
import { envVars } from "./app/config/env";
import { seedSuperAdmin } from "./app/utils/seedSuperAdmin";

let server: Server;


const startServer = async () => {
    try {
        await mongoose.connect(envVars.DB_URL)

        console.log("Connected to DB!!");

        server = app.listen(envVars.PORT, () => {
            console.log(`Server is listening to port ${envVars.PORT}`);
        });
    } catch (error) {
        console.log(error);
    }
}

(async () => {
    await startServer()
    await seedSuperAdmin()
})()

process.on("SIGTERM", () => {
    console.log("SIGTERM signal recieved... Server shutting down..");

    if (server) {
        server.close(() => {
            process.exit(1)
        });
    }

    process.exit(1)
})

process.on("SIGINT", () => {
    console.log("SIGINT signal recieved... Server shutting down..");

    if (server) {
        server.close(() => {
            process.exit(1)
        });
    }

    process.exit(1)
})


process.on("unhandledRejection", (err) => {
    console.log("Unhandled Rejecttion detected... Server shutting down..", err);

    if (server) {
        server.close(() => {
            process.exit(1)
        });
    }

    process.exit(1)
})

process.on("uncaughtException", (err) => {
    console.log("Uncaught Exception detected... Server shutting down..", err);

    if (server) {
        server.close(() => {
            process.exit(1)
        });
    }

    process.exit(1)
})

// Unhandler rejection error
// Promise.reject(new Error("I forgot to catch this promise"))

// Uncaught Exception Error
// throw new Error("I forgot to handle this local erro")


/**
 * unhandled rejection error
 * uncaught rejection error
 * signal termination sigterm
 */

```

### Update `user.controller.ts` at modules `src/app/modules/user/user.controller.ts`
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
//         // throw new Error("Fake eror")
//         // throw new AppError(httpStatus.BAD_REQUEST, "fake error")

//         // createUserFunction(req, res)

//     } catch (err: any) {
//         console.log(err);
//         next(err)
//     }
// }
const createUser = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const user = await UserServices.createUser(req.body)

    // res.status(httpStatus.CREATED).json({
    //     message: "User Created Successfully",
    //     user
    // })

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.CREATED,
        message: "User Created Successfully",
        data: user,
    })
})
const updateUser = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const userId = req.params.id;
    // const token = req.headers.authorization
    // const verifiedToken = verifyToken(token as string, envVars.JWT_ACCESS_SECRET) as JwtPayload

    const verifiedToken = req.user;

    const payload = req.body;
    const user = await UserServices.updateUser(userId, payload, verifiedToken)

    // res.status(httpStatus.CREATED).json({
    //     message: "User Created Successfully",
    //     user
    // })

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.CREATED,
        message: "User Updated Successfully",
        data: user,
    })
})

const getAllUsers = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const result = await UserServices.getAllUsers();

    // res.status(httpStatus.OK).json({
    //     success: true,
    //     message: "All Users Retrieved Successfully",
    //     data: users
    // })
    sendResponse(res, {
        success: true,
        statusCode: httpStatus.CREATED,
        message: "All Users Retrieved Successfully",
        data: result.data,
        meta: result.meta
    })
})

// function => try-catch catch => req-res function

export const UserControllers = {
    createUser,
    getAllUsers,
    updateUser
}

// route matching -> controller -> service -> model -> DB
```

### Create `interfaces/index.d.ts` at `app` folder
```ts
import { JwtPayload } from "jsonwebtoken";


declare global {
    namespace Express {
        interface Request {
            user: JwtPayload
        }
    }
}
```
