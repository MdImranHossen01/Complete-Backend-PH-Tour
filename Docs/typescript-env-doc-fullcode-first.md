# 🌱 TypeScript Environment Variable Loader

This document shows how to **load and validate environment variables** in a TypeScript project using `dotenv`, and explains the code line-by-line for beginners. You can reuse this in any backend Node.js/TypeScript project.

---

## ✅ Full Code First

```ts
import dotenv from "dotenv";

dotenv.config()

interface EnvConfig {
    PORT: string,
    DB_URL: string,
    NODE_ENV: "development" | "production"
}

const loadEnvVariables = (): EnvConfig => {
    const requiredEnvVariables: string[] = ["PORT", "DB_URL", "NODE_ENV"];

    requiredEnvVariables.forEach(key => {
        if (!process.env[key]) {
            throw new Error(`Missing require environment variabl ${key}`)
        }
    })

    return {
        PORT: process.env.PORT as string,
        // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
        DB_URL: process.env.DB_URL!,
        NODE_ENV: process.env.NODE_ENV as "development" | "production"
    }
}

export const envVars = loadEnvVariables()
```

---

## 🧠 Line-by-Line Explanation

### 1. Import dotenv
```ts
import dotenv from "dotenv";
```
Loads the `dotenv` package, which lets you use a `.env` file to define environment variables.

---

### 2. Configure dotenv
```ts
dotenv.config()
```
Tells dotenv to load variables from `.env` into `process.env`.

---

### 3. Define EnvConfig Interface
```ts
interface EnvConfig {
    PORT: string,
    DB_URL: string,
    NODE_ENV: "development" | "production"
}
```
Defines what types your environment variables must be.

---

### 4. Function to Load & Validate
```ts
const loadEnvVariables = (): EnvConfig => { ... }
```
Declares a function that will load and return all required environment variables in the correct type.

---

### 5. Required Environment Keys
```ts
const requiredEnvVariables: string[] = ["PORT", "DB_URL", "NODE_ENV"];
```
These keys must exist in the `.env` file.

---

### 6. Loop to Check for Missing Variables
```ts
requiredEnvVariables.forEach(key => {
    if (!process.env[key]) {
        throw new Error(...)
    }
})
```
Throws an error if any required variable is missing.

---

### 7. Return Config Object
```ts
return {
    PORT: process.env.PORT as string,
    DB_URL: process.env.DB_URL!,
    NODE_ENV: process.env.NODE_ENV as "development" | "production"
}
```
Returns all env values as a correctly typed object.

---

### 8. Export the Result
```ts
export const envVars = loadEnvVariables()
```
Executes the loader and makes the config available elsewhere in the app.

---

## 🧪 Example .env File

```
PORT=5000
DB_URL=mongodb://localhost:27017/myapp
NODE_ENV=development
```

---

## ♻️ Reuse This

Save this code in a file like `config/env.ts` and import `envVars` wherever you need access to your env settings.

---