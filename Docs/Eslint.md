## 🧹Eslint [Module: 25.11](https://web.programming-hero.com/level2-batch-5/video/level2-batch-5-25-11-setting-up-es-lint-and-fix-errors-using-commands)
[Read Documentation](https://typescript-eslint.io/getting-started)

- Install eslint 
```
npm install --save-dev eslint @eslint/js typescript typescript-eslint
```
- create an `eslint.config.mjs` config file in the root of your project, and populate it with the
```javascript
eslint.config.mjs
---
// @ts-check

import eslint from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  eslint.configs.recommended,
  //   tseslint.configs.recommended
  tseslint.configs.strict,
  tseslint.configs.stylistic,
  {
    rules : {
        "no-console" : "warn"
    }
  }
);

```
- **Add** `"lint": "npx eslint ./src",` script to `jackage.json` 
- **Running ESLint:** Open a terminal to the root of your project and run the following command
```
npm run lint
```
---