# 👤 Mongoose User Model with Schema Definitions (Beginner Friendly)

This file defines the **MongoDB schema for a User** using **Mongoose** and TypeScript interfaces.

---

## ✅ Full Code

```ts
import { model, Schema } from "mongoose";
import { IAuthProvider, IsActive, IUser, Role } from "./user.interface";

const authProviderSchema = new Schema<IAuthProvider>({
    provider: { type: String, required: true },
    providerId: { type: String, required: true }
}, {
    versionKey: false,
    _id: false
})

const userSchema = new Schema<IUser>({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String },
    role: {
        type: String,
        enum: Object.values(Role),
        default: Role.USER
    },
    phone: { type: String },
    picture: { type: String },
    address: { type: String },
    isDeleted: { type: Boolean, default: false },
    isActive: {
        type: String,
        enum: Object.values(IsActive),
        default: IsActive.ACTIVE,
    },
    isVerified: { type: Boolean, default: false },
    auths: [authProviderSchema],
}, {
    timestamps: true,
    versionKey: false
})

export const User = model<IUser>("User", userSchema)
```

---

## 📘 Line-by-Line Explanation

### 📦 `import { model, Schema } from "mongoose";`
- Imports core Mongoose classes to define and create models.
- `Schema` defines the structure of documents.
- `model` creates the actual MongoDB model based on that schema.

### 📤 `import { IAuthProvider, IsActive, IUser, Role } from "./user.interface";`
- Imports interfaces and enums from a separate `user.interface.ts` file for type safety.

---

## 🧩 Auth Provider Sub-Schema

### 🔸 `const authProviderSchema = new Schema<IAuthProvider>({...})`
- A **sub-schema** for storing third-party login data like Google or credentials.

#### Fields:
- `provider`: The method of login (e.g., `"Google"` or `"Credential"`).
- `providerId`: Unique ID from the login provider.

#### Options:
- `versionKey: false`: Disables `__v` field.
- `_id: false`: Prevents automatic `_id` creation for sub-documents.

---

## 🧱 Main User Schema

### 🔹 `const userSchema = new Schema<IUser>({...})`
Defines how a User document is structured in MongoDB.

### Fields:

| Field         | Type     | Details |
|---------------|----------|---------|
| `name`        | String   | Required name |
| `email`       | String   | Required, must be unique |
| `password`    | String   | Optional, not required for OAuth users |
| `role`        | Enum     | One of `SUPER_ADMIN`, `ADMIN`, `USER`, or `GUIDE` |
| `phone`       | String   | Optional phone number |
| `picture`     | String   | Optional profile picture URL |
| `address`     | String   | Optional physical address |
| `isDeleted`   | Boolean  | If the user is soft-deleted (default: false) |
| `isActive`    | Enum     | Can be `ACTIVE`, `INACTIVE`, or `BLOCKED` |
| `isVerified`  | Boolean  | Email or profile verified status (default: false) |
| `auths`       | Array    | List of auth provider objects using `authProviderSchema` |

### Schema Options:
- `timestamps: true`: Automatically adds `createdAt` and `updatedAt` fields.
- `versionKey: false`: Removes the default versioning field `__v`.

---

## 🏁 Final Model Export

```ts
export const User = model<IUser>("User", userSchema)
```
- Creates the `User` model from the `userSchema`.
- Exports it so it can be used to interact with the `users` collection in the database.

---

## 📝 Summary

- This file defines how a user is stored in the database.
- Uses interfaces and enums for type safety and clean code.
- Handles both normal and third-party login methods.
- Adds helpful options like timestamps and enum validation.