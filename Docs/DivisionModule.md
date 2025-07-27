# Division Module

### Create Necessary Folder  `📂division` and files `📜division.interface.ts`,  `📜division.model.ts`

```
📦modules
 ┣ 📂division
 ┃ ┣ 📜division.interface.ts
 ┃ ┗ 📜division.model.ts
```

### Add code `📜division.interface.ts`
```ts 
export interface IDivision {
    name: string;
    slug: string;
    thumbnail?: string;
    description?: string
}

/**
 * division name = Chattogram Division
 * 
 * slug = chattogram-division
 * 
 * /:id => /efwl432qgyqahwe
 * 
 * /:id => 
 * 
 * /:slug => /division/chattogram-division
 * 
 */
```

### Add code `📜division.model.ts`
```ts
import { model, Schema } from "mongoose";
import { IDivision } from "./division.interface";


const divisionSchema = new Schema<IDivision>({
    name: { type: String, required: true, unique: true },
    slug: { type: String, unique: true },
    thumbnail: { type: String },
    description: { type: String }
}, {
    timestamps: true
})

export const Division = model<IDivision>("Division", divisionSchema)
```













