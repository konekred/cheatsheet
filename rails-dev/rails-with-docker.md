#### local installation


# How to create a rails app inside docker
- create the app folder

- create *Gemfile* file

- add to top line
```
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }
```

- add rails gem
```
gem 'rails', '~> 5.2.2'
```

- create *Gemfile.lock* file

- create *Dockerfile* file
```
FROM ruby:2.4.5

RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs

WORKDIR /usr/src/app

ADD Gemfile ./Gemfile
ADD Gemfile.lock ./Gemfile.lock

RUN bundle install

ADD . .
```

- create *docker-compose.yml* file
```
version: '3'
services:
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ABC12abc
      MYSQL_DATABASE: grist_db
      MYSQL_USER: grist_db_user
      MYSQL_PASSWORD: ABC12abc
    ports:
      - '3307:3306'
  app:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - '.:/usr/src/app'
    ports:
      - '3001:3000'
    depends_on:
      - db
    links:
      - db
    environment:
      DB_NAME: grist_db
      DB_USER: root
      DB_PASS: ABC12abc
      DB_HOST: db

```

- setup the containers and create rails app
```
docker-compose run app rails new . --force --database=mysql --skip-bundle
```

- update *database.yml*
```
default: &default
  adapter: mysql2
  encoding: utf8
  pool: 5
  database: <%= ENV['DB_NAME'] %>
  username: <%= ENV['DB_USER'] %>
  password: <%= ENV['DB_PASS'] %>
  host: <%= ENV['DB_HOST'] %>

development:
  <<: *default

test:
  <<: *default

production:
  <<: *default
```

- update *docker-compose.yml* setup environment variables for database name,user,pass,host

- build the image
```
docker-compose build
```

- run the app
```
docker-compose up
```




