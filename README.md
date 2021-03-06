# ruby-graphql-server-example

[![Heroku deployed](https://img.shields.io/badge/Heroku-Deployed-8363A8.svg)](https://ruby-graphql-server-example.herokuapp.com)

> An example [GraphQL](http://graphql.org/) server implementation in Ruby

This project uses [Sinatra](http://www.sinatrarb.com/), [Sequel](http://sequel.jeremyevans.net/) and the [`graphql-ruby`](https://github.com/rmosolgo/graphql-ruby) gem to create a [Postgres](https://www.postgresql.org)-backed [GraphQL](http://graphql.org/) server in Ruby. It implements the HTTP protocol outlined in the [GraphQL HTTP best practices](http://graphql.org/learn/serving-over-http/).

You can [try it out on Heroku](https://ruby-graphql-server-example.herokuapp.com).

## Contents

* [Requirements](#requirements)
* [Installation](#installation)
* [Usage](#usage)
* [Data model](#data-model)
* [Example queries](#example-queries)
* [GraphiQL](#graphiql)
* [Development](#development)

## Requirements

* [Ruby](https://www.ruby-lang.org/en/)
* [Bundler](http://bundler.io/)
* [PostgreSQL](https://www.postgresql.org/)

## Installation

```
bundle install
```

## Usage

Setup a local database:

```
bundle exec rake db:migrate
```

Add some seed data:

```
bundle exec rake db:seed
```

Start the server:

```
bundle exec ruby app.rb
```

This will start a server running with a GraphQL endpoint at [http://localhost:4567/graphql](http://localhost:4567/graphql) and the [GraphiQL tool](#graphiql) at [http://localhost:4567](http://localhost:4567).

## Data model

The data model represents a simple video-on-demand service that displays shows, seasons and episodes.

```graphql
type Show {
  id: ID!
  title: String!
  seasons: [Season]!
  episodes: [Episode]!
}

type Season {
  id: ID!
  title: String!
  number: Int!
  episodes: [Episode]!
}

type Episode {
  id: ID!
  title: String!
  number: Int!
  show: Show!
  season: Season!
}
```

## Example queries

Let's create a simple GraphQL query in a `query.graphql` file that returns all of the shows:

```graphql
{
  shows {
    id
    title
  }
}
```

We can run that query against the server using `curl`:

```bash
curl -X POST http://localhost:4567/graphql \
  -H 'Content-Type: application/graphql' \
  -d @query.graphql | jq .
```

> Here we're using [`jq`](https://stedolan.github.io/jq/) (`brew install jq`) to get a pretty-printed JSON output

The output of the `curl` command will look like:

```json
{
  "data": {
    "shows": [
      {
        "id": "b013pqnm",
        "title": "The Great British Bake-Off"
      },
      {
        "id": "b0071b63",
        "title": "The Apprentice"
      },
      {
        "id": "b062r9t5",
        "title": "People Just Do Nothing"
      }
    ]
  }
}
```

We can then modify the query to return the nested episodes in each show:

```graphql
{
  shows {
    id
    title
    episodes {
      id
      number
      title
    }
  }
}
```

Which returns:

```json
{
  "data": {
    "shows": [
      {
        "id": "b013pqnm",
        "title": "The Great British Bake-Off",
        "episodes": [
          {
            "id": "b07r246c",
            "number": 1,
            "title": "Cake Week"
          },
          {
            "id": "b07tj10j",
            "number": 2,
            "title": "Biscuit Week"
          },
          {
            "id": "b07v324h",
            "number": 3,
            "title": "Bread Week"
          }
        ]
      },
      {
        "id": "b0071b63",
        "title": "The Apprentice",
        "episodes": [
          {
            "id": "b07ynzyc",
            "number": 1,
            "title": "Collectables"
          },
          {
            "id": "b07zfh1b",
            "number": 2,
            "title": "Advertising - Jeans"
          },
          {
            "id": "b0806vqc",
            "number": 3,
            "title": "Corporate Candy"
          }
        ]
      },
      {
        "id": "b062r9t5",
        "title": "People Just Do Nothing",
        "episodes": [
          {
            "id": "p043m903",
            "number": 1,
            "title": "Dubplate"
          },
          {
            "id": "b07tl80w",
            "number": 2,
            "title": "Record Deal"
          },
          {
            "id": "p045280q",
            "number": 3,
            "title": "Court Case"
          }
        ]
      }
    ]
  }
}
```

We can use arguments on the `episodes` field to return only the latest episode in each show:

```graphql
{
  shows {
    id
    title
    episodes(limit: 1, sortDirection: DESC) {
      id
      number
      title
    }
  }
}
```

```json
{
  "data": {
    "shows": [
      {
        "id": "b013pqnm",
        "title": "The Great British Bake-Off",
        "episodes": [
          {
            "id": "b07v324h",
            "number": 3,
            "title": "Bread Week"
          }
        ]
      },
      {
        "id": "b0071b63",
        "title": "The Apprentice",
        "episodes": [
          {
            "id": "b0806vqc",
            "number": 3,
            "title": "Corporate Candy"
          }
        ]
      },
      {
        "id": "b062r9t5",
        "title": "People Just Do Nothing",
        "episodes": [
          {
            "id": "p045280q",
            "number": 3,
            "title": "Court Case"
          }
        ]
      }
    ]
  }
}
```

We can query for a particular show:

```graphql
{
  show(id: "b013pqnm") {
    id
    title
  }
}
```

```json
{
  "data": {
    "show": {
      "id": "b013pqnm",
      "title": "The Great British Bake-Off"
    }
  }
}
```

And then retrieve its entire hierarchy, including seasons and episodes:

```graphql
{
  show(id: "b013pqnm") {
    id
    title,
    seasons {
      id
      title
      number
      episodes {
        id
        title
        number
      }
    }
  }
}
```

```json
{
  "data": {
    "show": {
      "id": "b013pqnm",
      "title": "The Great British Bake-Off",
      "seasons": [
        {
          "id": "b07r2pr0",
          "title": "Series 7",
          "number": 7,
          "episodes": [
            {
              "id": "b07r246c",
              "title": "Cake Week",
              "number": 1
            },
            {
              "id": "b07tj10j",
              "title": "Biscuit Week",
              "number": 2
            },
            {
              "id": "b07v324h",
              "title": "Bread Week",
              "number": 3
            }
          ]
        }
      ]
    }
  }
}
```

We can also a retrieve an episode by its ID:

```graphql
{
  episode(id: "p043m903") {
    id
    number
    title
  }
}
```

```json
{
  "data": {
    "episode": {
      "id": "p043m903",
      "number": 1,
      "title": "Dubplate"
    }
  }
}
```

And return its season and show:

```graphql
{
  episode(id: "p043m903") {
    id
    number
    title
    season {
      id
      title
    }
    show {
      id
      title
    }
  }
}
```

```json
{
  "data": {
    "episode": {
      "id": "p043m903",
      "number": 1,
      "title": "Dubplate",
      "season": {
        "id": "p043m8g0",
        "title": "Series 3"
      },
      "show": {
        "id": "b062r9t5",
        "title": "People Just Do Nothing"
      }
    }
  }
}
```

## GraphiQL

The server includes the [GraphiQL](https://github.com/graphql/graphiql) tool.

You can use it to explore the API in your browser by visiting [http://localhost:4567/](http://localhost:4567).

The GraphiQL installation follows the [official example](https://github.com/graphql/graphiql/tree/master/example).

## Development

To use [Shotgun](https://github.com/rtomayko/shotgun) to reload the server in development:

```
bundle exec shotgun
```

All SQL queries are logged by default in development. This is disabled when the `RACK_ENV` environment variable is missing or set to `production`.
