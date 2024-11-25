# Turing Solutions Dashboard
This project contains the dashboard for Turing Solutions.

## CMS

Running the CMS locally can be done with the following 2 commands:

```
pnpm i
```

```
pnpm dev
```

# TLDR
Out of the box payloadcms (v3) with Dockerfile did not work. These things changed so it works:

- Installed sqlite adapter 'pnpm i @payloadcms/db-sqlite'
- Altered Dockerfile slightly (see Dockerfile in this project) (AND DONT USED ALPINE BECAUSE OF 'linux-x64-musl' error, instead e.g. use node-slim);
- Docker compose uses the image that is build via the dockerfile, instead of locally using the app as volume;
- executed 'pnpm install @libsql/client' for resolving cannot find module 'libsql';
- Add 'output: 'standalone',' to payload.config.ts

### via docker compose

Create Docker image
```
docker build -t payload-cms-demo-app:v1 .
```

```
docker compose up 
```

go to 
```
http://localhost:3000/admin
```


## Attributes

- **Database**: SQLite

## Generate models
The models/data scheme can be generated with the following command:
```
payload generate:types
```
This command generates TypeScript classes based on the CMS configuration. These classes can be used in the site project(s).

## Installation process
This section describe the steps that were taken to setup this project. The base of the project was generated via npx (25-nov-2024). 
This came with a default Dockerfile and docker-compose.yaml. These files did not seem to work when creating a payload cms 3 docker container and using sqlite. 

The base of the template is made for using mongo db. Using mongodb means you need a "real" database and for some cases sqlite will be more suitable.  

See the rest of this file for all things that changed comparing to the default project.

### Create project (option sqlite adapter)
```
npx create-payload-app
```

### add the required Payload packages to your project
```
pnpm i payload @payloadcms/next @payloadcms/richtext-lexical sharp graphql
```


### Install Payload SQLite adapter
```
pnpm i @payloadcms/db-sqlite
```


### Extend next.config
Add the following code to the next.config.mjs for creating a standalone build output that can be run in a docker container independent:
```
  output: 'standalone',
  reactStrictMode: true,  
```

File looks something like this then:
```
import { withPayload } from '@payloadcms/next/withPayload'

/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
  reactStrictMode: true,  
}

export default withPayload(nextConfig)
```

### Optionally needed
For some reason I ran into some issues. These were fixed by:

- delete .next folder;
- delete node_modules folder;
- ```$ pnp run build```

After executing the command, the .next folder should contain a folder named "standalone".

## Dockerfile and Docker compose
The payloadCMS project came with a docker-compose file out of the box. This file hosted a nodejs environment and mounted the app via a volume to the container. Some changes were made in the Dockerfile and Docker compose to make this work. 

Changes in Dockerfile:
- Explicitly use pnpm instead of the if-statement structure;
- Remove line for copying public directory (because it did not exist??);
- remove yarn reference in copy lockfile;
- Add a .dockerignore to the project with at least "node_modules" in it.
- Due to a specific bug the node alpine version cant be used. node alpine was replaced with 'node:20-slim' that does work;



Changes in docker-compose:
- Use explicit container build instead of mounting local app;