# Docker, KeystoneJS, MongoDB and Yarn

Without a Yarn specific _keystone-generator_ it is difficult to scaffold a new KeystoneJS project without _npm_. For simplicity, we will generate the initial project
which uses `npm`, then delete the node_modules dir and finally run `yarn`.

## Generate a New KeystoneJS Project

The recommended way to generate a KeystoneJS project is via the Yeoman KeystoneJS generator, _generator-keystone_. The generator requires some manual intervention and thus the first evolution of our dockerfile will enable us to ssh in and intervene accordingly:

```dockerfile
FROM node:6.11.3

RUN curl -o- -L https://yarnpkg.com/install.sh | bash
RUN yarn global add yo generator-keystone

ENV HOME=/home/app

WORKDIR $HOME
```

And we'll run everything with docker-compose.yml

```yaml
version: '2'
services:
  my-service:
    build:
      context: .
      dockerfile: ./dockerfile.keystone
```

1. Build it
```
$ docker-compose build
```

2. Run it, we mount the current directory and all the project files generated in the container will sync with our local directory.
```
$ docker run -it --rm -v $PWD:/home/app {IMAGE_ID} bash
```

3. Generate the project, this will generate the starter project for us. Watch the logs after executing the following command, as soon the `creates...` finish, kill the command with `ctrl+c`. We don't want our dependencies via npm!
```
$ yo keystone
```

4. Remove any `npm` installed assets
```
$ npm cache clean && rm -r {node_modules/,.node-gyp/,.config}
```

5. Kill it
```
ctrl + d
```

## Install dependencies with Yarn

Now we need to update our dockerfile to re-install the project dependencies, albeit with Yarn this time! Also, we can ditch Yeoman.

```dockerfile
FROM node:6.11.3

RUN curl -o- -L https://yarnpkg.com/install.sh | bash

# Be a good guy
RUN useradd --user-group --create-home --shell /bin/false app

ENV WORKDIR=/home/app/

COPY ./package.json $WORKDIR
RUN chown -R app:app $WORKDIR/*

USER app
WORKDIR $WORKDIR
RUN yarn

EXPOSE 3000
```

Update the docker-compose.

`- /home/app/node_modules` this line ensures we don't shadow the node_modules directory on the container when we mount our volume. Remove it and see what happens.

```yaml
version: '2'
services:
  my-service:
    build:
      context: .
      dockerfile: ./dockerfile.keystone
    command: node keystone
    volumes:
      - .:/home/app
      - /home/app/node_modules
    expose:
      - "3000"
    ports:
      - "3000:3000"
```

1. Build it
```
$ docker-compose build --no-cache && docker-compose up
```

You should notice at this stage, we're missing a mongo service! Let's include it.

## Add a mongo service

Update the docker-compose.yml to include our Mongo service

```yaml
version: '2'
services:
  my-service:
    build:
      context: .
      dockerfile: ./dockerfile.keystone
    depends_on:
      - my-db-service
    command: node keystone
    volumes:
      - .:/home/app
      - /home/app/node_modules
    expose:
      - "3000"
    ports:
      - "3000:3000"
  my-db-service:
    image: mongo:latest
```

1. We'll need to let Keystone where the database is. Add the following line to .env

```
MONGO_URI=my-db-service
```

2. Bring the services up
```
$ docker-compose up -d
```

3. Clear up any dangling images

```
docker rmi $(docker images --quiet --filter "dangling=true")
```

