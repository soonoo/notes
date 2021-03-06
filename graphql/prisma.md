# Prisma

## Packages

For local development, the [`prisma`](http://npmjs.com/package/prisma) package should be installed.

```bash
npm i -g prisma
```

Additionally, the [`graphql-cli`](https://www.npmjs.com/package/graphql-cli) package is useful for bootstrapping a project.

```bash
npm i -g graphql-cli
```

## Bootstrapping

The `graphql-cli`'s `create` command is used to bootstrap a project using one of a number of prepared setups.

```bash
graphql create <dir>
```

The file structure will look something like this (but varies based on which preset you use):

```bash
./
  database/
    datamodel.graphql
    prisma.yml
    seed.graphql
  src/
    generated/
      prisma.graphql
    resolvers/
      Mutations/
        auth.js
        post.js
      AuthPayload.js
      index.js
      Query.js
      Subscription.js
    index.js
    schema.graphql
    utils.js
  .env
  .graphqlconfig.yml
  package.json
  README.md
```

#### Default Service

This initiates with a "public" Prisma service (as opposed to one linked to your Prisma account). It also automatically deploys to Prisma, which isn't necessarily ideal. `prisma delete` can be used to delete the default service.

```bash
prisma delete
```

#### Prisma Secret

You may need to include the Prisma secret in your `prisma.yml`. `graphql create` creates a generic secret in the `.env` file, so you can import it in the `prisma.yml`, but you should also change it to something more secure (random).

When using a secret, the `GraphQlServer` should also include the secret.

```js
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
  context: req => ({
    ...req,
    db: new Prisma({
      endpoint: process.env.PRISMA_ENDPOINT,
      debug: true,
      secret: process.env.PRISMA_SECRET, // !!!!
    }),
  }),
});
```

#### TypeScript

For TypeScript projects, a `graphql codegen` call should be added to the post-deploy hooks so that the TypeScript definitions are correctly generated after a deploy.

```bash
# prisma.yml
endpoint: ${env:PRISMA_ENDPOINT}

datamodel: database/datamodel.graphql

secret: ${env:PRISMA_SECRET}

hooks:
  post-deploy:
    - graphql get-schema --project database
    - graphql codegen
```

#### Private Service

A new service can be generated by calling `prisma deploy --new`. However, this will overwrite the endpoint in `prisma.yml` with a hard-coded string of the new service. You can either keep that as is, or (preferably) add the new service's location to the `.env` and restore the `PRISMA_ENDPOINT` import.
