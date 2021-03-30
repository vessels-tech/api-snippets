# api-snippets

API Snippets makes it easy to work with Mojaloop's APIs, by breaking down the definitions into reusable, composable chunks.

## Swagger UI Api Preview

https://docs.mojaloop.io/api-snippets/

## Usage

Install the snippet library
```bash
  npm install @mojaloop/api-snippets --save-dev
```

Install the reference resolving library
```bash
  npm install @redocly/openapi-cli --save-dev
```

Modify swagger file to reference `api-snippets`.

ex.
```yaml
Money:
  $ref: /path/to/node_modules/@mojaloop/api-snippets/fspiop/v1_1/openapi3/components/schemas/Money.yaml
```

Render and resolve the references.
```bash
  openapi bundle --output openapi.yaml --ext yaml openapi_template.yaml
```

Validate the result file.
```bash
  swagger-cli validate api_render.yaml
```

## Quick Demonstration
![demonstration](./docs/demonstration.gif "Demonstration")

## Swagger-UI

The snippets specification is previewable using swagger-ui. Swagger-UI files
are found in `docs/dist/`. Github pages uses the master branch `docs/` folder
to build the page found at https://docs.mojaloop.io/api-snippets/

## Dev Tools

To create snippets the tool found at https://redoc.ly/docs/cli/#installation-and-usage
was used to break up large specification files. Some manual edits to the paths
are needed to format the files.

TODO: Add more detailed instructions for this workflow.
- `npm install -g @redocly/openapi-cli`
- `openapi split fspiop-rest-v1.1-openapi3-snippets.yaml --outDir ./fspiop/v1_1`

## Questions

1) Do we need to support both Swagger 2.0 and Open Api 3?

## DTO TypeScript definitions for snippets
> [Data Transfer Object](https://en.wikipedia.org/wiki/Data_transfer_object) description


The reason to have DTO is to allow separation between data transfer and data access phases.

From OpenAPI definition perspective `api-snippets` module allows building api.yaml definitions using a structured set of building blocks - `the snippets`.

From microservices implementation point of view there is a need to have similar mechanism to allow declaring aggregated api data transfer types using basic types. These basic types reflect api-snippets definitions at implementation language level of TypeScript

### Example of payload definition

Let assume we want to define a new type to represent the http request POST payload body with two properties: `requestId` and `amount` using already defined in `api-snippets` basic types: `CorrelationId` and `Money`


```typescript
import { Schemas } from '@mojaloop/api-snippets/v1_0'

interface ExampleTransferRequest {
  requestId: Schemas.CorrelationId
  amount: Schemas.Money
}
```

### structure of `@mojaloop/api-snippets` TypeScript type system module


`api-snippets` OpenAPI definitions are grouped in three sections/folders with corresponding Javascript sub-modules

| OpenApi                      | TypeScript            |
|------------------------------|-----------------------|
| v1.0/openapi3/               | v1_0                  |
| v1.1/openapi3/               | v1_1                  |
| thirdparty/opeanapi3         | thirdparty            |


The same structure is kept in TypesScript module, but with some handy shortcuts: there are no `openapi3/schemas` sub-folders/sub-modules

To use the types defined for `CorrelationId` and `Money` snippets
- `v1.0/openapi3/schemas/CorrelationId.yaml`
- `v1.0/openapi3/schemas/Money.yaml`

You can import them in a such short way:
```typescript
import { Schemas } from `@mojaloop/api-snippets/v1_0`

interface ExampleTransferRequest {
  requestId: Schemas.CorrelationId
  amount: Schemas.Money
}
```

You can also have access to full `openapi` interface auto generated by [openapi-typescript](https://www.npmjs.com/package/openapi-typescript) including `paths` & `components`
```typescript
import { openapi } from `@mojaloop/api-snippets/v1_0`

interface ExampleTransferRequest {
  requestId: openapi.components['schemas']['CorrelationId']
  amount: openapi.components['schemas']['Money']
}
```

### DTO auto generation

For development of api-snippets module please use this `mantra`
```bash
npm run build
```

It will generate OpenAPI specifications from snippets, then generate DTO from these snippets and finally generate javascript code

### Auto generation for micro services
When developing a micro service, to generate interfaces from your own OpenAPI specification custom types, please use
```bash
npx openapi-typescript src/interface/api.yaml --output src/interface/openapi.ts
```

The `src/interface/api.yaml` should be final OpenAPI specification, not a template one.

`openapi -typescript` cli is installed in your system as `@mojaloop/api-snippets` dependency

By convention we are keeping micro service OpenAPI specification in `src/interface` folder
### DTO type aliasing
Type aliasing could be very handy when working with `src/interface/openapi.ts` file. Suggested pattern is to create a new file `src/interface/api.ts` and declare aliases inside:

```typescript
import { components } from './openapi.ts'

// reexport openapi if needed
export * as openapi from './openapi.ts'

// let define some aliases for schemas
export namespace Schemas {
  export type CorrelationId = components['schemas']['CorrelationId']
  export type Money = components['schemas']['Money']
  export type MyCustomRequest = components['schemas']['MyCustomRequest']
}
```

So later in you implementation code you can refer to interfaces via these aliases:

```typescript
import { Schemas } from 'src/interface/api.ts'

let request: Schemas.MyCustomRequest = { ... }
```

### DTO usage examples
please check in `src/example.ts` for further reading

### DTO linting
to lint generated files use:
```bash
npm run lint
```

the `npm run build:dto` script is already executing linting with auto-fixing

there is pre-commit hook setup witch do a linting for staged files.
### DTO testing
- testing of DTO declarations
