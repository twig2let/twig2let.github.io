# Create a New Rails Project With Docker

Sometimes you may need to communicate between services defined across multiple compositions I.E services that Docker will start on isolated networks. To enable our services to talk one another as if they had all been defined in one docker-compose.yml we can leverage Docker's `user-defined networks`, in this case the [`Bridged`](https://docs.docker.com/engine/userguide/networking/#bridge-networks) network.

___

## Create Rails Project

1. Start a Ruby container.

```sh
docker run -it -v $PWD:/opt ruby:latest bash
```

2. Install Rails.

```sh
gem install Rails
```

3. Create a new Rails project.

```sh
cd opt && rails new {PROJECT_NAME}
```

## Configure Rails Project

1. Uncomment the mini_racer gem in the Gemfile.

2. Add Postgres Gem to the Gemfile

```Ruby
gem 'pg', '~> 0.21.0'
```

3. Replace contents of config/database.yml with:

```yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: 5
  username: user
  password: password
  host: db

development:
  <<: *default
  database: {PROJECT_NAME}-dev

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: {PROJECT_NAME}-test

```

4. In bin/setup, replace:

```Ruby
  system! 'bin/rails restart'
```

with:

```Ruby
  system! 'bin/rails server'
```

5. Fire it up!

```sh
cd docker && docker-compose up
```
