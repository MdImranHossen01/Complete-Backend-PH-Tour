# Tour Management System - Backend Documentation

This project was developed as part of the Apollo Level 2 Web Development course. Below are the links to all important project-related documentation.

---

### 📋 Project Planning & Analysis

-   **Requirement Analysis:** [Detailed description of the project's requirements and features](https://docs.google.com/document/d/1XRN18ClObPMJGKl7CfZFBLeZuJf2h8JOBc_crzuWtT4/edit?tab=t.0)
-   **System Workflow:** [Visual diagram of the system's workflow](https://gitmind.com/app/docs/mzkbj5o2)

---

### 🗃️ Database Design

-   **Data Modelling:** [Description of the system's data model and schema](https://docs.google.com/document/d/1NSELQ7_jUx4xLGchef4HT3_9YqDWrdzUUGMZWmD2lY0/edit?tab=t.0)
-   **ER Diagram:** [Entity-Relationship Diagram (ERD)](https://drive.google.com/file/d/1ASphx7B6gHIKPiiiNf3AB_ZTvdYRBsQz/view?usp=sharing)

---

### ⚙️ API & Source Code

-   **API Endpoints:** [List and description of all project API endpoints](https://docs.google.com/document/d/1HysoioRCpSsGpSz8JQZRGii9GNkpdx0pEX-zHP2p334/edit?tab=t.0)
-   **GitHub Repository:** [The complete source code for the project](https://github.com/Apollo-Level2-Web-Dev/ph-tour-management-system-backend)

---
---

# Setup project & Github

- Create a folder for the project and rename. 
- Open In Vs Code
- In terminal write `git init`
![git init](image.png)
- Create `.gitignore` file in project folder

```javascript 
node_modules
```
**Change git branch**
- type `git checkout -b development` in terminal to switch development branch
![Change branch to development](image-1.png)
- *run `git add .` and then `git commit -m"your meaningful message"` in terminal to commit in git.*
![git commit](image-2.png)
---
### Setup Project
- run `npm init -y` in terminal for package.json
- install typescript dev dependency `npm i -D typescript` in terminal
- initialize typescript `tsc --init` in terminal
- find `"rootDir": "./src", ` in `tsconfig.json` file
- find `"outDir": "./dist",` in `tsconfig.json` file
- install mandatory packages `npm i express mongoose zod jsonwebtoken cors dotenv`
- install mandatory dev-dependencies `npm i -D ts-node-dev @types/express @types/cors @types/dotenv @types/jsonwebtoken`
- create 📦src and 📦dist folder in root
- create `app.ts` and `server.ts` file and `app` folder in
```
📦src
 ┣ 📂app
 ┣ 📜app.ts
 ┗ 📜server.ts
 ```
 ---
 ### Moduler MVC patern:-
 - create Moduler MVC patern file and folder for tour and user:-
 ```
 📦src
 ┣ 📂app
 ┃ ┗ 📂modules
 ┃ ┃ ┣ 📂tour
 ┃ ┃ ┃ ┣ 📜tour.controller.ts
 ┃ ┃ ┃ ┣ 📜tour.interface.ts
 ┃ ┃ ┃ ┗ 📜tour.model.ts
 ┃ ┃ ┗ 📂user
 ┃ ┃ ┃ ┣ 📜user.controller.ts
 ┃ ┃ ┃ ┣ 📜user.interface.ts
 ┃ ┃ ┃ ┗ 📜user.model.ts
 ┣ 📜app.ts
 ┗ 📜server.ts
 ``` 
---