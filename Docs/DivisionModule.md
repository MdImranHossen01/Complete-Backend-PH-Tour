# Division and Tour [Module 29.9-30.12](https://web.programming-hero.com/level2-batch-5/video/level2-batch-5-30-1-create-slug-during-create-and-update-of-division-and-tour)

### Create  Folder  `📂division` and `📂tour`

```
📦modules
 ┣ 📂division
 ┃ ┣ 📜division.controller.ts
 ┃ ┣ 📜division.interface.ts
 ┃ ┣ 📜division.model.ts
 ┃ ┣ 📜division.route.ts
 ┃ ┣ 📜division.service.ts
 ┃ ┗ 📜division.validation.ts
 ┣ 📂tour
 ┃ ┣ 📜tour.constant.ts
 ┃ ┣ 📜tour.controller.ts
 ┃ ┣ 📜tour.interface.ts
 ┃ ┣ 📜tour.model.ts
 ┃ ┣ 📜tour.route.ts
 ┃ ┣ 📜tour.service.ts
 ┃ ┗ 📜tour.validation.ts

```

## Division files codes
---


```ts

// src/app/modules/division/division.controller.ts

import { Request, Response } from "express";
import { catchAsync } from "../../utils/catchAsync";
import { sendResponse } from "../../utils/sendResponse";
import { DivisionService } from "./division.service";

const createDivision = catchAsync(async (req: Request, res: Response) => {
    const result = await DivisionService.createDivision(req.body);
    sendResponse(res, {
        statusCode: 201,
        success: true,
        message: "Division created",
        data: result,
    });
});

const getAllDivisions = catchAsync(async (req: Request, res: Response) => {
    const result = await DivisionService.getAllDivisions();
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: "Divisions retrieved",
        data: result.data,
        meta: result.meta,
    });
});
const getSingleDivision = catchAsync(async (req: Request, res: Response) => {
    const slug = req.params.slug
    const result = await DivisionService.getSingleDivision(slug);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: "Divisions retrieved",
        data: result.data,
    });
});

const updateDivision = catchAsync(async (req: Request, res: Response) => {
    const id = req.params.id;

    const result = await DivisionService.updateDivision(id, req.body);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: "Division updated",
        data: result,
    });
});

const deleteDivision = catchAsync(async (req: Request, res: Response) => {
    const result = await DivisionService.deleteDivision(req.params.id);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: "Division deleted",
        data: result,
    });
});

export const DivisionController = {
    createDivision,
    getAllDivisions,
    getSingleDivision,
    updateDivision,
    deleteDivision,
};
```
```ts
// src/app/modules/division/division.interface.ts

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
```ts
// src/app/modules/division/division.model.ts

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


divisionSchema.pre("save", async function (next) {
    if (this.isModified("name")) {
        const baseSlug = this.name.toLowerCase().split(" ").join("-")
        let slug = `${baseSlug}-division`

        let counter = 0;
        while (await Division.exists({ slug })) {
            slug = `${slug}-${counter++}` // dhaka-division-2
        }

        this.slug = slug;
    }
    next()
})

divisionSchema.pre("findOneAndUpdate", async function (next) {
    const division = this.getUpdate() as Partial<IDivision>

    if (division.name) {
        const baseSlug = division.name.toLowerCase().split(" ").join("-")
        let slug = `${baseSlug}-division`

        let counter = 0;
        while (await Division.exists({ slug })) {
            slug = `${slug}-${counter++}` // dhaka-division-2
        }

        division.slug = slug
    }

    this.setUpdate(division)

    next()
})

export const Division = model<IDivision>("Division", divisionSchema)
```
```ts
// src/app/modules/division/division.route.ts

import { Router } from "express";
import { checkAuth } from "../../middlewares/checkAuth";
import { validateRequest } from "../../middlewares/validateRequest";
import { Role } from "../user/user.interface";
import { DivisionController } from "./division.controller";
import {
    createDivisionSchema,
    updateDivisionSchema,
} from "./division.validation";

const router = Router()

router.post(
    "/create",
    checkAuth(Role.ADMIN, Role.SUPER_ADMIN),
    validateRequest(createDivisionSchema),
    DivisionController.createDivision
);
router.get("/", DivisionController.getAllDivisions);
router.get("/:slug", DivisionController.getSingleDivision)
router.patch(
    "/:id",
    checkAuth(Role.ADMIN, Role.SUPER_ADMIN),
    validateRequest(updateDivisionSchema),
    DivisionController.updateDivision
);
router.delete("/:id", checkAuth(Role.ADMIN, Role.SUPER_ADMIN), DivisionController.deleteDivision);

export const DivisionRoutes = router
```
```ts
// src/app/modules/division/division.service.ts

import { IDivision } from "./division.interface";
import { Division } from "./division.model";

const createDivision = async (payload: IDivision) => {

    const existingDivision = await Division.findOne({ name: payload.name });
    if (existingDivision) {
        throw new Error("A division with this name already exists.");
    }


    // const baseSlug = payload.name.toLowerCase().split(" ").join("-")
    // let slug = `${baseSlug}-division`

    // let counter = 0;
    // while (await Division.exists({ slug })) {
    //     slug = `${slug}-${counter++}` // dhaka-division-2
    // }

    // payload.slug = slug;

    const division = await Division.create(payload);

    return division
};

const getAllDivisions = async () => {
    const divisions = await Division.find({});
    const totalDivisions = await Division.countDocuments();
    return {
        data: divisions,
        meta: {
            total: totalDivisions
        }
    }
};
const getSingleDivision = async (slug: string) => {
    const division = await Division.findOne({ slug });
    return {
        data: division,
    }
};



const updateDivision = async (id: string, payload: Partial<IDivision>) => {

    const existingDivision = await Division.findById(id);
    if (!existingDivision) {
        throw new Error("Division not found.");
    }

    const duplicateDivision = await Division.findOne({
        name: payload.name,
        _id: { $ne: id },
    });

    if (duplicateDivision) {
        throw new Error("A division with this name already exists.");
    }

    // if (payload.name) {
    //     const baseSlug = payload.name.toLowerCase().split(" ").join("-")
    //     let slug = `${baseSlug}-division`

    //     let counter = 0;
    //     while (await Division.exists({ slug })) {
    //         slug = `${slug}-${counter++}` // dhaka-division-2
    //     }

    //     payload.slug = slug
    // }

    const updatedDivision = await Division.findByIdAndUpdate(id, payload, { new: true, runValidators: true })

    return updatedDivision

};

const deleteDivision = async (id: string) => {
    await Division.findByIdAndDelete(id);
    return null;
};

export const DivisionService = {
    createDivision,
    getAllDivisions,
    getSingleDivision,
    updateDivision,
    deleteDivision,
};
```
```ts
// src/app/modules/division/division.validation.ts
import { z } from "zod";

export const createDivisionSchema = z.object({
    name: z.string().min(1),
    thumbnail: z.string().optional(),
    description: z.string().optional(),
});

export const updateDivisionSchema = z.object({
    name: z.string().min(1).optional(),
    thumbnail: z.string().optional(),
    description: z.string().optional(),
});
```

## Tour files codes

```ts
// src/app/modules/tour/tour.constant.ts
export const tourSearchableFields = ["title", "description", "location"]

```
```ts
// src/app/modules/tour/tour.controller.ts

import { Request, Response } from 'express';
import { catchAsync } from '../../utils/catchAsync';
import { sendResponse } from '../../utils/sendResponse';
import { TourService } from './tour.service';

const createTour = catchAsync(async (req: Request, res: Response) => {
    const result = await TourService.createTour(req.body);
    sendResponse(res, {
        statusCode: 201,
        success: true,
        message: 'Tour created successfully',
        data: result,
    });
});

const getAllTours = catchAsync(async (req: Request, res: Response) => {

    const query = req.query
    const result = await TourService.getAllTours(query as Record<string, string>);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: 'Tours retrieved successfully',
        data: result.data,
        meta: result.meta,
    });
});

const updateTour = catchAsync(async (req: Request, res: Response) => {

    const result = await TourService.updateTour(req.params.id, req.body);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: 'Tour updated successfully',
        data: result,
    });
});

const deleteTour = catchAsync(async (req: Request, res: Response) => {
    const { id } = req.params;
    const result = await TourService.deleteTour(id);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: 'Tour deleted successfully',
        data: result,
    });
});
const getAllTourTypes = catchAsync(async (req: Request, res: Response) => {
    const result = await TourService.getAllTourTypes();
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: 'Tour types retrieved successfully',
        data: result,
    });
});


const createTourType = catchAsync(async (req: Request, res: Response) => {
    const { name } = req.body;
    const result = await TourService.createTourType(name);
    sendResponse(res, {
        statusCode: 201,
        success: true,
        message: 'Tour type created successfully',
        data: result,
    });
});

const updateTourType = catchAsync(async (req: Request, res: Response) => {
    const { id } = req.params;
    const { name } = req.body;
    const result = await TourService.updateTourType(id, name);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: 'Tour type updated successfully',
        data: result,
    });
});
const deleteTourType = catchAsync(async (req: Request, res: Response) => {
    const { id } = req.params;
    const result = await TourService.deleteTourType(id);
    sendResponse(res, {
        statusCode: 200,
        success: true,
        message: 'Tour type deleted successfully',
        data: result,
    });
});

export const TourController = {
    createTour,
    createTourType,
    getAllTourTypes,
    deleteTourType,
    updateTourType,
    getAllTours,
    updateTour,
    deleteTour,
};
```
```ts
// src/app/modules/tour/tour.interface.ts

import { Types } from "mongoose";

export interface ITourType {
    name: string;
}
export interface ITour {
    title: string;
    slug: string;
    description?: string;
    images?: string[];
    location?: string;
    costFrom?: number;
    startDate?: Date
    departureLocation?: string;
    arrivalLocation?: string;
    endDate?: Date;
    included?: string[];
    excluded?: string[]
    amenities?: string[];
    tourPlan?: string[];
    maxGuest?: number;
    minAge?: number;
    division: Types.ObjectId
    tourType: Types.ObjectId
}
```
```ts
// src/app/modules/tour/tour.model.ts
import { model, Schema } from "mongoose";
import { ITour, ITourType } from "./tour.interface";

const tourTypeSchema = new Schema<ITourType>({
    name: { type: String, required: true, unique: true }
}, {
    timestamps: true
})

export const TourType = model<ITourType>("TourType", tourTypeSchema)

const tourSchema = new Schema<ITour>({
    title: { type: String, required: true },
    slug: { type: String, unique: true },
    description: { type: String },
    images: { type: [String], default: [] },
    location: { type: String },
    costFrom: { type: Number },
    startDate: { type: Date },
    endDate: { type: Date },
    departureLocation: { type: String },
    arrivalLocation: { type: String },
    included: { type: [String], default: [] },
    excluded: { type: [String], default: [] },
    amenities: { type: [String], default: [] },
    tourPlan: { type: [String], default: [] },
    maxGuest: { type: Number },
    minAge: { type: Number },
    division: {
        type: Schema.Types.ObjectId,
        ref: "Division",
        required: true
    },
    tourType: {
        type: Schema.Types.ObjectId,
        ref: "TourType",
        required: true
    }
}, {
    timestamps: true
})

tourSchema.pre("save", async function (next) {

    if (this.isModified("title")) {
        const baseSlug = this.title.toLowerCase().split(" ").join("-")
        let slug = `${baseSlug}`

        let counter = 0;
        while (await Tour.exists({ slug })) {
            slug = `${slug}-${counter++}` // dhaka-division-2
        }

        this.slug = slug;
    }
    next()
})

tourSchema.pre("findOneAndUpdate", async function (next) {
    const tour = this.getUpdate() as Partial<ITour>

    if (tour.title) {
        const baseSlug = tour.title.toLowerCase().split(" ").join("-")
        let slug = `${baseSlug}`


        let counter = 0;
        while (await Tour.exists({ slug })) {
            slug = `${slug}-${counter++}` // dhaka-division-2
        }

        tour.slug = slug
    }

    this.setUpdate(tour)

    next()
})

export const Tour = model<ITour>("Tour", tourSchema)
```
```ts
// src/app/modules/tour/tour.route.ts
import express from "express";
import { checkAuth } from "../../middlewares/checkAuth";
import { validateRequest } from "../../middlewares/validateRequest";
import { Role } from "../user/user.interface";
import { TourController } from "./tour.controller";
import {
    createTourTypeZodSchema,
    createTourZodSchema,
    updateTourZodSchema,
} from "./tour.validation";

const router = express.Router();

/* ------------------ TOUR TYPE ROUTES -------------------- */
router.get("/tour-types", TourController.getAllTourTypes);

router.post(
    "/create-tour-type",
    checkAuth(Role.ADMIN, Role.SUPER_ADMIN),
    validateRequest(createTourTypeZodSchema),
    TourController.createTourType
);

router.patch(
    "/tour-types/:id",
    checkAuth(Role.ADMIN, Role.SUPER_ADMIN),
    validateRequest(createTourTypeZodSchema),
    TourController.updateTourType
);

router.delete("/tour-types/:id", checkAuth(Role.ADMIN, Role.SUPER_ADMIN), TourController.deleteTourType);

/* --------------------- TOUR ROUTES ---------------------- */
router.get("/", TourController.getAllTours);

router.post(
    "/create",
    checkAuth(Role.ADMIN, Role.SUPER_ADMIN),
    validateRequest(createTourZodSchema),
    TourController.createTour
);

router.patch(
    "/:id",
    checkAuth(Role.ADMIN, Role.SUPER_ADMIN),
    validateRequest(updateTourZodSchema),
    TourController.updateTour
);

router.delete("/:id", checkAuth(Role.ADMIN, Role.SUPER_ADMIN), TourController.deleteTour);




export const TourRoutes = router
```
```ts
// src/app/modules/tour/tour.service.ts

import { QueryBuilder } from "../../utils/QueryBuilder";
import { tourSearchableFields } from "./tour.constant";
import { ITour, ITourType } from "./tour.interface";
import { Tour, TourType } from "./tour.model";

const createTour = async (payload: ITour) => {
    const existingTour = await Tour.findOne({ title: payload.title });
    if (existingTour) {
        throw new Error("A tour with this title already exists.");
    }

    // const baseSlug = payload.title.toLowerCase().split(" ").join("-")
    // let slug = `${baseSlug}`

    // let counter = 0;
    // while (await Tour.exists({ slug })) {
    //     slug = `${slug}-${counter++}` // dhaka-division-2
    // }

    // payload.slug = slug;

    const tour = await Tour.create(payload)

    return tour;
};

// const getAllToursOld = async (query: Record<string, string>) => {
//     console.log(query);
//     const filter = query
//     const searchTerm = query.searchTerm || "";
//     const sort = query.sort || "-createdAt";
//     const page = Number(query.page) || 1
//     const limit = Number(query.limit) || 10
//     const skip = (page - 1) * limit

//     //field fitlering
//     const fields = query.fields?.split(",").join(" ") || ""

//     //old field => title,location
//     //new fields => title location

//     // delete filter["searchTerm"]
//     // delete filter["sort"]



//     for (const field of excludeField) {
//         // eslint-disable-next-line @typescript-eslint/no-dynamic-delete
//         delete filter[field]
//     }

//     console.log(filter);



//     const searchQuery = {
//         $or: tourSearchableFields.map(field => ({ [field]: { $regex: searchTerm, $options: "i" } }))
//     }

//     // [remove][remove][remove](SKip)[][][][][][]

//     // [][][][][](limit)[remove][remove][remove][remove]

//     // 1 page => [1][1][1][1][1][1][1][1][1][1] skip = 0 limit =10
//     // 2 page => [1][1][1][1][1][1][1][1][1][1]=>skip=>[2][2][2][2][2][2][2][2][2][2]<=limit skip = 10 limit =10
//     // 3 page => [1][1][1][1][1][1][1][1][1][1]=>skip=>[2][2][2][2][2][2][2][2][2][2]<=limit skip = 20 limit = 10

//     // skip = (page -1) * 10 = 30

//     // ?page=3&limit=10

//     // const tours = await Tour.find(searchQuery).find(filter).sort(sort).select(fields).skip(skip).limit(limit);

//     const filterQuery = Tour.find(filter)

//     const tours = filterQuery.find(searchQuery)

//     const allTours = await tours.sort(sort).select(fields).skip(skip).limit(limit)

//     // location = Dhaka
//     // search = Golf
//     const totalTours = await Tour.countDocuments();
//     // const totalPage = 21/10 = 2.1 => ciel(2.1) => 3
//     const totalPage = Math.ceil(totalTours / limit)

//     const meta = {
//         page: page,
//         limit: limit,
//         total: totalTours,
//         totalPage: totalPage,
//     }
//     return {
//         data: allTours,
//         meta: meta
//     }
// };

const getAllTours = async (query: Record<string, string>) => {


    const queryBuilder = new QueryBuilder(Tour.find(), query)

    const tours = await queryBuilder
        .search(tourSearchableFields)
        .filter()
        .sort()
        .fields()
        .paginate()

    // const meta = await queryBuilder.getMeta()

    const [data, meta] = await Promise.all([
        tours.build(),
        queryBuilder.getMeta()
    ])


    return {
        data,
        meta
    }
};


const updateTour = async (id: string, payload: Partial<ITour>) => {

    const existingTour = await Tour.findById(id);

    if (!existingTour) {
        throw new Error("Tour not found.");
    }

    // if (payload.title) {
    //     const baseSlug = payload.title.toLowerCase().split(" ").join("-")
    //     let slug = `${baseSlug}`

    //     let counter = 0;
    //     while (await Tour.exists({ slug })) {
    //         slug = `${slug}-${counter++}` // dhaka-division-2
    //     }

    //     payload.slug = slug
    // }

    const updatedTour = await Tour.findByIdAndUpdate(id, payload, { new: true });

    return updatedTour;
};

const deleteTour = async (id: string) => {
    return await Tour.findByIdAndDelete(id);
};

const createTourType = async (payload: ITourType) => {
    const existingTourType = await TourType.findOne({ name: payload.name });

    if (existingTourType) {
        throw new Error("Tour type already exists.");
    }

    return await TourType.create({ name });
};
const getAllTourTypes = async () => {
    return await TourType.find();
};
const updateTourType = async (id: string, payload: ITourType) => {
    const existingTourType = await TourType.findById(id);
    if (!existingTourType) {
        throw new Error("Tour type not found.");
    }

    const updatedTourType = await TourType.findByIdAndUpdate(id, payload, { new: true });
    return updatedTourType;
};
const deleteTourType = async (id: string) => {
    const existingTourType = await TourType.findById(id);
    if (!existingTourType) {
        throw new Error("Tour type not found.");
    }

    return await TourType.findByIdAndDelete(id);
};

export const TourService = {
    createTour,
    createTourType,
    deleteTourType,
    updateTourType,
    getAllTourTypes,
    getAllTours,
    updateTour,
    deleteTour,
};
```
```ts
// src/app/modules/tour/tour.validation.ts
import { z } from "zod";



export const createTourZodSchema = z.object({
    title: z.string(),
    description: z.string().optional(),
    location: z.string().optional(),
    costFrom: z.number().optional(),
    startDate: z.string().optional().optional(),
    endDate: z.string().optional().optional(),
    tourType: z.string(),// <- changed here
    included: z.array(z.string()).optional(),
    excluded: z.array(z.string()).optional(),
    amenities: z.array(z.string()).optional(),
    tourPlan: z.array(z.string()).optional(),
    maxGuest: z.number().optional(),
    minAge: z.number().optional(),
    division: z.string(),
    departureLocation: z.string().optional(),
    arrivalLocation: z.string().optional()
});

export const updateTourZodSchema = z.object({
    title: z.string().optional(),
    description: z.string().optional(),
    location: z.string().optional(),
    costFrom: z.number().optional(),
    startDate: z.string().optional().optional(),
    endDate: z.string().optional().optional(),
    tourType: z.string().optional(),// <- changed here
    included: z.array(z.string()).optional(),
    excluded: z.array(z.string()).optional(),
    amenities: z.array(z.string()).optional(),
    tourPlan: z.array(z.string()).optional(),
    maxGuest: z.number().optional(),
    minAge: z.number().optional(),
    departureLocation: z.string().optional(),
    arrivalLocation: z.string().optional()
});

export const createTourTypeZodSchema = z.object({
    name: z.string(),
});
```


## Update `index.ts` file at `src/app/routes/index.ts`

```ts

import { Router } from "express"
import { AuthRoutes } from "../modules/auth/auth.route"
import { UserRoutes } from "../modules/user/user.route"
import { DivisionRoutes } from "../modules/division/division.route"
import { TourRoutes } from "../modules/tour/tour.route"

export const router = Router()

const moduleRoutes = [
    {
        path: "/user",
        route: UserRoutes
    },
    {
        path: "/auth",
        route: AuthRoutes
    },
    {
        path: "/division",
        route: DivisionRoutes
    },
    {
        path: "/tour",
        route: TourRoutes
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


## Create file `constants.ts` at `src/app/constants.ts`

```ts
export const excludeField = ["searchTerm", "sort", "fields", "page", "limit"]
```

### Create `QueryBuilder.ts` at  `src/app/utils/QueryBuilder.ts`
```ts

import { Query } from "mongoose";
import { excludeField } from "../constants";

export class QueryBuilder<T> {
    public modelQuery: Query<T[], T>;
    public readonly query: Record<string, string>

    constructor(modelQuery: Query<T[], T>, query: Record<string, string>) {
        this.modelQuery = modelQuery;
        this.query = query;
    }


    filter(): this {
        const filter = { ...this.query }

        for (const field of excludeField) {
            // eslint-disable-next-line @typescript-eslint/no-dynamic-delete
            delete filter[field]
        }

        this.modelQuery = this.modelQuery.find(filter) // Tour.find().find(filter)

        return this;
    }

    search(searchableField: string[]): this {
        const searchTerm = this.query.searchTerm || ""
        const searchQuery = {
            $or: searchableField.map(field => ({ [field]: { $regex: searchTerm, $options: "i" } }))
        }
        this.modelQuery = this.modelQuery.find(searchQuery)
        return this
    }

    sort(): this {

        const sort = this.query.sort || "-createdAt";

        this.modelQuery = this.modelQuery.sort(sort)

        return this;
    }
    fields(): this {

        const fields = this.query.fields?.split(",").join(" ") || ""

        this.modelQuery = this.modelQuery.select(fields)

        return this;
    }
    paginate(): this {

        const page = Number(this.query.page) || 1
        const limit = Number(this.query.limit) || 10
        const skip = (page - 1) * limit

        this.modelQuery = this.modelQuery.skip(skip).limit(limit)

        return this;
    }

    build() {
        return this.modelQuery
    }

    async getMeta() {
        const totalDocuments = await this.modelQuery.model.countDocuments()

        const page = Number(this.query.page) || 1
        const limit = Number(this.query.limit) || 10

        const totalPage = Math.ceil(totalDocuments / limit)

        return { page, limit, total: totalDocuments, totalPage }
    }
}
```
















