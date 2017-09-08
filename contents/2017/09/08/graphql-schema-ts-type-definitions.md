GraphQL schema.json TypeScript type definitions
===

### Prerequisistes

```bash
$ npm i @types/graphql
```

### GraphQL schema.json TypeScript type definitions

Normal JSON file's `d.ts` [How to Import json into TypeScript](https://hackernoon.com/import-json-into-typescript-8d465beded79)

`graphql-schema-json.d.ts`

```ts
declare module "*/schema.json" {
    import { IntrospectionQuery } from "graphql";
    interface SchemaJson {
        data: IntrospectionQuery;
    }
    const value: SchemaJson;
    export = value;
}
```

When you put an above file and edit `tsconfig.json` appropriately, you can import `schema.json` in ts files like this.

```ts
import * as schemaJson from "path/2/schema.json";

console.dir(schemaJson.data);
```
