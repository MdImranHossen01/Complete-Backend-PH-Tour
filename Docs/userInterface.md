# 📄 User Interface & Role Model Explanation (TypeScript + Mongoose)

## ✅ Full Code

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
    provider: string;  // "Google", "Credential"
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
    isVerified?: string;
    role: Role;
    auths: IAuthProvider[]
    bookings?: Types.ObjectId[]
    guides?: Types.ObjectId[]
}
```

---

## 📘 Line-by-Line Explanation

### ✅ `import { Types } from "mongoose";`
- Imports the `Types` object from the Mongoose library.
- `Types.ObjectId` is used to reference MongoDB document IDs (used for `bookings` and `guides` later).

---

### 🔐 `export enum Role {...}`
Defines a TypeScript `enum` to strictly define allowed roles for a user:

- `SUPER_ADMIN`: Highest-level admin with full permissions.
- `ADMIN`: Administrative-level user.
- `USER`: Regular user with basic permissions.
- `GUIDE`: Specialized role, likely for users who guide or manage tasks.

---

### 🧾 `//auth providers`

```ts
/**
 * email, password 
 * google authentication
 */
```

Describes that users can authenticate via:
- Traditional `email/password`.
- External providers like `Google`.

---

### 🔐 `export interface IAuthProvider {...}`
Defines the structure for each auth provider attached to a user:
- `provider`: String value like `"Google"` or `"Credential"`.
- `providerId`: Unique ID for the provider (e.g., Google user ID or internal credential ID).

---

### 🟢 `export enum IsActive {...}`
Enumerates the user's account status:
- `ACTIVE`: The account is active and usable.
- `INACTIVE`: Temporarily deactivated.
- `BLOCKED`: Banned or restricted from access.

---

### 👤 `export interface IUser {...}`
Defines the full structure of a User document in MongoDB.

| Property      | Type                    | Description |
|---------------|-------------------------|-------------|
| `name`        | `string`                | Full name of the user |
| `email`       | `string`                | Email address |
| `password?`   | `string (optional)`     | Password (optional for OAuth users) |
| `phone?`      | `string (optional)`     | User's phone number |
| `picture?`    | `string (optional)`     | URL to profile picture |
| `address?`    | `string (optional)`     | Physical or mailing address |
| `isDeleted?`  | `string (optional)`     | Could be a flag or timestamp (not strongly typed here) |
| `isActive?`   | `IsActive (optional)`   | Status from the enum `IsActive` |
| `isVerified?` | `string (optional)`     | Possibly a verification flag or timestamp |
| `role`        | `Role`                  | User's role from the `Role` enum |
| `auths`       | `IAuthProvider[]`       | List of auth providers connected to the user |
| `bookings?`   | `Types.ObjectId[]`      | Array of booking references (linked documents) |
| `guides?`     | `Types.ObjectId[]`      | Array of guides related to the user |

---

## 📝 Summary

- This code defines structured data models for users in a MongoDB + Mongoose backend using TypeScript.
- Helps ensure strong typing, validation, and clean relationships (like references to other collections).
- Modular and scalable approach for user roles, authentication providers, and status handling.