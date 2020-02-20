---
published: true
title: 'Node.js Restful API template with TypeScript, Fastify and MongoDB'
cover_image: ''
description: 'Using wp-cli on Bash on Windows 10'
tags: node, fastify, typescript, mongodb
series:
canonical_url: 'https://worldwildweb.dev/node-js-restful-api-template-with-typescript-fastify-and-mongodb'
---

# Why

Have you recently started a new Node.js API project? Did you use some template or started the project from scratch?
I was asking the same questions myself and I was looking for minimal boilerplate for a while. There were so many options that it was hard to pick one.
Most of them are using Express.js, others are using ES5 or lack test setup.
So I decided to spin one on my own and reuse it in the future.

# How

My setup has the following characteristics:

## API

- Node version 10 or later
- TypeScript for obvious reasons
- Fastify for its asynchronous nature and being faster than Express or Restify
- Nodemon in development for watching for changes and restart the server

## Data

- MongoDB with Mongoose
- Docker for MongoDB service instead of installing it

## Tests

- Jest for being the de-facto in Node testing
- In memory Mongod server for easily mock the DB
- Coverall for coverage collector after Jest report is generated

## Code formatting and static analysis

- ESLint config
- Prettier config attached to the linter
- Editor config

## Documentation

- Swagger UI for API documentation
- Postman collections attached from testing the endpoints

## CI

- Continuous integration in Travis CI.
  Steps:

1. Install dependencies
2. Run tests
3. Collect coverage and pass it to Coverall

And thats it! I hope it's minimal enough.
Please share some ideas for improvement if you have any. I thought of API versioning but Fastify seems to support that out of the box.
API key authentication was also something I was considering, but I wasn't sure how exactly to implement it. If you have something in mind would love to discuss it in the comments.
Happy coding!
