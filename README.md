# Building a Server with TypeScript and GraphQL

## Introduction to TypeScript

## Introduction to GraphQL

### Types

- Defines types of data
- Possible types
  - Object
  - Scalat
  - Enums
  - Interfaces

### Schema

- Contract between client and server
- Defines types, arguments and directives
- Defines relations between objects
- Defines API behaviors

### Querying

- Fetch data
- Specify data
- Filter data

### Mutating

- Modify data on server
- Can do:
  - Create
  - Update
  - Delete

### Subscriptions

- Realtime updates
- No manual refresh


## Building a TypeScript and GraphQL server

### Setup

```json
{
  "name": "sample-project",
  "version": "1.0.0",
  "description": "",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "dev": "nodemon --watch 'src/**/*.ts' --exec 'ts-node' src/server.ts",
    "start": "node dist/index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "apollo-server-express": "^3.13.0",
    "express": "^4.19.2",
    "graphql": "^16.8.1"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/graphql": "^14.5.0",
    "@types/node": "^20.12.4",
    "nodemon": "^3.1.0",
    "ts-node": "^10.9.2",
    "typescript": "^5.4.3"
  }
}
```

### GraphQL schema

```ts
import { gql } from "apollo-server-express";

const Schema = gql`
  type Car {
    id: ID!
    carType: String!
    color: String!
    year: Int!
  }

  type Query {
    getCars: [Car!]!
  }

  type Mutation {
    addCar(carType: String!, color: String!, year: Int!): Car
  }
`;

export default Schema;
```

### GraphQL resolver

```ts
const resolvers = {
  Query: {
    getCars: () => cars,
  },

  Mutation: {
    addCar: (_: any, args: any) => {
      const newCar = {
        id: cars.length + 1,
        carType: args.carType,
        color: args.color,
        year: args.year,
      };
      cars.push(newCar);
      return newCar;
    },
  },
};
```

### Server

```ts
import { ApolloServer } from "apollo-server-express";
import Schema from "./schema";
import Resolvers from "./resolvers";
import express from "express";
import { ApolloServerPluginDrainHttpServer } from "apollo-server-core";
import http from "http";

async function startApolloServer(schema: any, resolvers: any) {
  const app = express();
  const httpServer = http.createServer(app);
  const server = new ApolloServer({
    typeDefs: schema,
    resolvers,
    //Attach GraphQL Functionality to Express Server
    plugins: [ApolloServerPluginDrainHttpServer({ httpServer })],
  }) as any;
  await server.start(); //start the GraphQL server.
  server.applyMiddleware({ app });
  await new Promise<void>(
    resolve => httpServer.listen({ port: 4000 }, resolve) //server runs on port 4000
  );
  console.log(`Server ready at http://localhost:4000${server.graphqlPath}`);
}
// Run Server and Pass Schema and Resolver
startApolloServer(Schema, Resolvers);
```