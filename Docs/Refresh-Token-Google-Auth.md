# Refresh Token-Google Auth

### Update `.env` file

```ts
PORT=5000
DB_URL=mongodb+srv://example_db_user_name:example_db_password@cluster0.edawjnd.mongodb.net/example-db?retryWrites=true&w=majority&appName=Cluster0
NODE_ENV=development

# JWT
JWT_ACCESS_SECRET=access_secret
JWT_ACCESS_EXPIRES=1d
JWT-REFRESH-SECRET=refresh_secret
JWT-REFRESH-EXPIRES=30

# BCRYPT
BCRYPT_SALT_ROUND=10

# SUPER ADMIN
SUPER_ADMIN_EMAIL=super@gmail.com
SUPER_ADMIN_PASSWORD=12345678

# Google
GOOGLE_CLIENT_SECRET=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8w66B
GOOGLE_CLIENT_ID=15454875970933-bndsaibhpgo478hgp986k5c45omchr.apps.googleusercontent.com
GOOGLE_CALLBACK_URL=http://localhost:5000/api/v1/auth/google/callback

# Express Session
EXPRESS_SESSION_SECRET=express-session

# Frontend URL
FRONTEND_URL=http://localhost:5173
```

### Update `env.ts` file at folder `config`
```ts
import dotenv from "dotenv";

dotenv.config()

interface EnvConfig {
    PORT: string,
    DB_URL: string,
    NODE_ENV: "development" | "production"
    BCRYPT_SALT_ROUND: string
    JWT_ACCESS_SECRET: string
    JWT_ACCESS_EXPIRES: string
    JWT_REFRESH_SECRET: string
    JWT_REFRESH_EXPIRES: string
    SUPER_ADMIN_EMAIL: string
    SUPER_ADMIN_PASSWORD: string
    GOOGLE_CLIENT_SECRET: string
    GOOGLE_CLIENT_ID: string
    GOOGLE_CALLBACK_URL: string
    EXPRESS_SESSION_SECRET: string
    FRONTEND_URL: string

}

const loadEnvVariables = (): EnvConfig => {
    const requiredEnvVariables: string[] = ["PORT", "DB_URL", "NODE_ENV", "BCRYPT_SALT_ROUND", "JWT_ACCESS_EXPIRES", "JWT_ACCESS_SECRET", "SUPER_ADMIN_EMAIL", "SUPER_ADMIN_PASSWORD", "JWT_REFRESH_SECRET", "JWT_REFRESH_EXPIRES", "GOOGLE_CLIENT_SECRET", "GOOGLE_CLIENT_ID", "GOOGLE_CALLBACK_URL", "EXPRESS_SESSION_SECRET", "FRONTEND_URL"];

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
        JWT_REFRESH_SECRET: process.env.JWT_REFRESH_SECRET as string,
        JWT_REFRESH_EXPIRES: process.env.JWT_REFRESH_EXPIRES as string,
        SUPER_ADMIN_EMAIL: process.env.SUPER_ADMIN_EMAIL as string,
        SUPER_ADMIN_PASSWORD: process.env.SUPER_ADMIN_PASSWORD as string,
        GOOGLE_CLIENT_SECRET: process.env.GOOGLE_CLIENT_SECRET as string,
        GOOGLE_CLIENT_ID: process.env.GOOGLE_CLIENT_ID as string,
        GOOGLE_CALLBACK_URL: process.env.GOOGLE_CALLBACK_URL as string,
        EXPRESS_SESSION_SECRET: process.env.EXPRESS_SESSION_SECRET as string,
        FRONTEND_URL: process.env.FRONTEND_URL as string

    }
}

export const envVars = loadEnvVariables()
```
### Update `auth.service.ts` file at folder `modules/auth`
```ts
/* eslint-disable @typescript-eslint/no-non-null-assertion */
import bcryptjs from "bcryptjs";
import httpStatus from "http-status-codes";
import { JwtPayload } from "jsonwebtoken";
import { envVars } from "../../config/env";
import AppError from "../../errorHelpers/AppError";
import { createNewAccessTokenWithRefreshToken, createUserTokens } from "../../utils/userTokens";
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
    // const jwtPayload = {
    //     userId: isUserExist._id,
    //     email: isUserExist.email,
    //     role: isUserExist.role
    // }
    // const accessToken = generateToken(jwtPayload, envVars.JWT_ACCESS_SECRET, envVars.JWT_ACCESS_EXPIRES)

    // const refreshToken = generateToken(jwtPayload, envVars.JWT_REFRESH_SECRET, envVars.JWT_REFRESH_EXPIRES)

    const userTokens = createUserTokens(isUserExist)

    // delete isUserExist.password;

    // eslint-disable-next-line @typescript-eslint/no-unused-vars
    const { password: pass, ...rest } = isUserExist.toObject()

    return {
        accessToken: userTokens.accessToken,
        refreshToken: userTokens.refreshToken,
        user: rest
    }

}
const getNewAccessToken = async (refreshToken: string) => {
    const newAccessToken = await createNewAccessTokenWithRefreshToken(refreshToken)

    return {
        accessToken: newAccessToken
    }

}
const resetPassword = async (oldPassword: string, newPassword: string, decodedToken: JwtPayload) => {

    const user = await User.findById(decodedToken.userId)

    const isOldPasswordMatch = await bcryptjs.compare(oldPassword, user!.password as string)
    if (!isOldPasswordMatch) {
        throw new AppError(httpStatus.UNAUTHORIZED, "Old Password does not match");
    }

    user!.password = await bcryptjs.hash(newPassword, Number(envVars.BCRYPT_SALT_ROUND))

    user!.save();


}

//user - login - token (email, role, _id) - booking / payment / booking / payment cancel - token 

export const AuthServices = {
    credentialsLogin,
    getNewAccessToken,
    resetPassword
}
```

### Create `userTokens.ts` file at folder `src/app/utils/userTokens.ts`

```ts

import httpStatus from "http-status-codes";
import { JwtPayload } from "jsonwebtoken";
import { envVars } from "../config/env";
import AppError from "../errorHelpers/AppError";
import { IsActive, IUser } from "../modules/user/user.interface";
import { User } from "../modules/user/user.model";
import { generateToken, verifyToken } from "./jwt";

export const createUserTokens = (user: Partial<IUser>) => {
    const jwtPayload = {
        userId: user._id,
        email: user.email,
        role: user.role
    }
    const accessToken = generateToken(jwtPayload, envVars.JWT_ACCESS_SECRET, envVars.JWT_ACCESS_EXPIRES)

    const refreshToken = generateToken(jwtPayload, envVars.JWT_REFRESH_SECRET, envVars.JWT_REFRESH_EXPIRES)


    return {
        accessToken,
        refreshToken
    }
}

export const createNewAccessTokenWithRefreshToken = async (refreshToken: string) => {

    const verifiedRefreshToken = verifyToken(refreshToken, envVars.JWT_REFRESH_SECRET) as JwtPayload


    const isUserExist = await User.findOne({ email: verifiedRefreshToken.email })

    if (!isUserExist) {
        throw new AppError(httpStatus.BAD_REQUEST, "User does not exist")
    }
    if (isUserExist.isActive === IsActive.BLOCKED || isUserExist.isActive === IsActive.INACTIVE) {
        throw new AppError(httpStatus.BAD_REQUEST, `User is ${isUserExist.isActive}`)
    }
    if (isUserExist.isDeleted) {
        throw new AppError(httpStatus.BAD_REQUEST, "User is deleted")
    }

    const jwtPayload = {
        userId: isUserExist._id,
        email: isUserExist.email,
        role: isUserExist.role
    }
    const accessToken = generateToken(jwtPayload, envVars.JWT_ACCESS_SECRET, envVars.JWT_ACCESS_EXPIRES)

    return accessToken
}
```

### Update `auth.route.ts` file at folder `src/app/modules/auth/auth.route.ts`

```ts
import { NextFunction, Request, Response, Router } from "express";
import passport from "passport";
import { checkAuth } from "../../middlewares/checkAuth";
import { Role } from "../user/user.interface";
import { AuthControllers } from "./auth.controller";

const router = Router()

router.post("/login", AuthControllers.credentialsLogin)
router.post("/refresh-token", AuthControllers.getNewAccessToken)
router.post("/logout", AuthControllers.logout)
router.post("/reset-password", checkAuth(...Object.values(Role)), AuthControllers.resetPassword)


//  /booking -> /login -> succesful google login -> /booking frontend
// /login -> succesful google login -> / frontend
router.get("/google", async (req: Request, res: Response, next: NextFunction) => {
    const redirect = req.query.redirect || "/"
    passport.authenticate("google", { scope: ["profile", "email"], state: redirect as string })(req, res, next)
})

// api/v1/auth/google/callback?state=/booking
router.get("/google/callback", passport.authenticate("google", { failureRedirect: "/login" }), AuthControllers.googleCallbackController)

export const AuthRoutes = router;
```

### Update `auth.controller.ts` file at folder `src/app/modules/auth/auth.controller.ts`
```ts
/* eslint-disable @typescript-eslint/no-unused-vars */
import { NextFunction, Request, Response } from "express"
import httpStatus from "http-status-codes"
import { JwtPayload } from "jsonwebtoken"
import { envVars } from "../../config/env"
import AppError from "../../errorHelpers/AppError"
import { catchAsync } from "../../utils/catchAsync"
import { sendResponse } from "../../utils/sendResponse"
import { setAuthCookie } from "../../utils/setCookie"
import { createUserTokens } from "../../utils/userTokens"
import { AuthServices } from "./auth.service"

const credentialsLogin = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const loginInfo = await AuthServices.credentialsLogin(req.body)

    // res.cookie("accessToken", loginInfo.accessToken, {
    //     httpOnly: true,
    //     secure: false
    // })


    // res.cookie("refreshToken", loginInfo.refreshToken, {
    //     httpOnly: true,
    //     secure: false,
    // })

    setAuthCookie(res, loginInfo)

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.OK,
        message: "User Logged In Successfully",
        data: loginInfo,
    })
})
const getNewAccessToken = catchAsync(async (req: Request, res: Response, next: NextFunction) => {
    const refreshToken = req.cookies.refreshToken;
    if (!refreshToken) {
        throw new AppError(httpStatus.BAD_REQUEST, "No refresh token recieved from cookies")
    }
    const tokenInfo = await AuthServices.getNewAccessToken(refreshToken as string)

    // res.cookie("accessToken", tokenInfo.accessToken, {
    //     httpOnly: true,
    //     secure: false
    // })

    setAuthCookie(res, tokenInfo);

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.OK,
        message: "New Access Token Retrived Successfully",
        data: tokenInfo,
    })
})
const logout = catchAsync(async (req: Request, res: Response, next: NextFunction) => {

    res.clearCookie("accessToken", {
        httpOnly: true,
        secure: false,
        sameSite: "lax"
    })
    res.clearCookie("refreshToken", {
        httpOnly: true,
        secure: false,
        sameSite: "lax"
    })

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.OK,
        message: "User Logged Out Successfully",
        data: null,
    })
})
const resetPassword = catchAsync(async (req: Request, res: Response, next: NextFunction) => {

    const newPassword = req.body.newPassword;
    const oldPassword = req.body.oldPassword;
    const decodedToken = req.user

    await AuthServices.resetPassword(oldPassword, newPassword, decodedToken as JwtPayload);

    sendResponse(res, {
        success: true,
        statusCode: httpStatus.OK,
        message: "Password Changed Successfully",
        data: null,
    })
})
const googleCallbackController = catchAsync(async (req: Request, res: Response, next: NextFunction) => {

    let redirectTo = req.query.state ? req.query.state as string : ""

    if (redirectTo.startsWith("/")) {
        redirectTo = redirectTo.slice(1)
    }

    // /booking => booking , => "/" => ""
    const user = req.user;

    if (!user) {
        throw new AppError(httpStatus.NOT_FOUND, "User Not Found")
    }

    const tokenInfo = createUserTokens(user)

    setAuthCookie(res, tokenInfo)

    // sendResponse(res, {
    //     success: true,
    //     statusCode: httpStatus.OK,
    //     message: "Password Changed Successfully",
    //     data: null,
    // })

    res.redirect(`${envVars.FRONTEND_URL}/${redirectTo}`)
})

export const AuthControllers = {
    credentialsLogin,
    getNewAccessToken,
    logout,
    resetPassword,
    googleCallbackController
}
```

- install
```
npm i cookie-parser
npm i -D @types/cookie-parser
```
### update `app.ts`
```ts
import cookieParser from "cookie-parser";
import cors from "cors";
import express, { Request, Response } from "express";
import expressSession from "express-session";
import passport from "passport";
import { envVars } from "./app/config/env";
import "./app/config/passport";
import { globalErrorHandler } from "./app/middlewares/globalErrorHandler";
import notFound from "./app/middlewares/notFound";
import { router } from "./app/routes";

const app = express()


app.use(expressSession({
    secret: envVars.EXPRESS_SESSION_SECRET,
    resave: false,
    saveUninitialized: false
}))
app.use(passport.initialize())
app.use(passport.session())
app.use(cookieParser())
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

### Create `/setCookie.ts` file at folder `src/app/utils/setCookie.ts`

```ts
import { Response } from "express";

export interface AuthTokens {
    accessToken?: string;
    refreshToken?: string;
}

export const setAuthCookie = (res: Response, tokenInfo: AuthTokens) => {
    if (tokenInfo.accessToken) {
        res.cookie("accessToken", tokenInfo.accessToken, {
            httpOnly: true,
            secure: false
        })
    }

    if (tokenInfo.refreshToken) {
        res.cookie("refreshToken", tokenInfo.refreshToken, {
            httpOnly: true,
            secure: false,
        })
    }
}
```

### Update `checkAuth.ts` file at folder `src/app/middlewares/checkAuth.ts`

```ts
import { NextFunction, Request, Response } from "express";
import { JwtPayload } from "jsonwebtoken";
import { envVars } from "../config/env";
import AppError from "../errorHelpers/AppError";
import { verifyToken } from "../utils/jwt";
import { User } from "../modules/user/user.model";
import httpStatus from "http-status-codes"
import { IsActive } from "../modules/user/user.interface";

export const checkAuth = (...authRoles: string[]) => async (req: Request, res: Response, next: NextFunction) => {

    try {
        const accessToken = req.headers.authorization;

        if (!accessToken) {
            throw new AppError(403, "No Token Recieved")
        }


        const verifiedToken = verifyToken(accessToken, envVars.JWT_ACCESS_SECRET) as JwtPayload

        const isUserExist = await User.findOne({ email: verifiedToken.email })

        if (!isUserExist) {
            throw new AppError(httpStatus.BAD_REQUEST, "User does not exist")
        }
        if (isUserExist.isActive === IsActive.BLOCKED || isUserExist.isActive === IsActive.INACTIVE) {
            throw new AppError(httpStatus.BAD_REQUEST, `User is ${isUserExist.isActive}`)
        }
        if (isUserExist.isDeleted) {
            throw new AppError(httpStatus.BAD_REQUEST, "User is deleted")
        }

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

















































