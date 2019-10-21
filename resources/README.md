# Learning GraphQL Project
      
- [Resolvers best practices](https://medium.com/paypal-engineering/graphql-resolvers-best-practices-cd36fdbcef55)
- [The approaches to writing resolvers](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3)
- [Nexus Library](https://www.prisma.io/blog/introducing-graphql-nexus-code-first-graphql-server-development-ll6s1yy5cxl5)

### Introduction to GraphQL

*Resource: https://graphql.org/learn/*

GraphQL is a query language. A GraphQL service is created by defining types and fields on those types, then providing **functions** for each field on each type.

An example of a schema would be

```
type Query {
  me: User
}

type User {
  id: ID
  name: String
}
```

Along with the functions for each field

```
function Query_me(request) {
  return request.auth.user;
}

function User_name(user) {
  return user.getName();
}
```

An example of the query:

```
{
  me {
    name
  }
}
```

Would produce the JSON result:

```
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```

#### Fields

In GraphQL field can also refer to Objects. You can make a *sub-selection* of a field for that object. GraphQL queries can traverse related objects and their fields, letting clients fetch lots of related data in on request.

Query:

```
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

Result:

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

#### Arguments

Unlike REST, in GraphQL you have the ability to pass arguments to fields, you can even pass arguments into scalar fields, to implemenent data transformation in the server, instead on on every client separately.

Query

```
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

Result

```
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

#### Aliases

Since you cannot directly query for the same field with different arguements, you need to uses *aliases* that let you rename the result of a field into anything we want.

Query

```
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

Result

```
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

#### Fragments

If we have a complicated query a way to optimize and avoid duplication would be to use *fragments*. Fragments allow you to construct sets of fields, and the include them in queries where we need to.

Query

```
{
  one: hero(episode: EMPIRE) {
    ...fieldGroup
  }
  two: hero(episode: JEDI) {
    ...fieldGroup
  }
  three: human(id: 1001){
    ...fieldGroup
  }
}

fragment fieldGroup on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

Result

```
{
  "data": {
    "one": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "two": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    },
    "three": {
      "name": "Darth Vader",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Wilhuff Tarkin"
        }
      ]
    }
  }
}
```

It is also possible to use variables inside fragments.

We can pass the `$first` variable and then use it to show the number of friends connections to return.

```
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

#### Operation Name

Doing shorthand syntax is useful however when dealing with production services it is best practice to use **Operation type** and **Operation Name**.

The operation type is either a query, mutation, or subscription.

Shorthand syntax

```
{
  hero {
    name
    friends {
      name
    }
  }
}
```

Syntax with operation type and operation name

```
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

#### Variables

In order to support dynamic manipulation of query string at runtime, rather than doing manipulation of the query in the client-side, GraphQL supports `variables`.

In order to start working with variables, we need to do three things:

1. Replace the static value in the query with `$variableName`
2. Declare `$variableName` as one of the variables accepted by the query
3. Pass `variableName: value` in the separate, transport-specific (usually JSON) variables dictionary

Query

```
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

Variables

```
{
  "episode": "JEDI"
}
```

Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

We can also define *default values* to our variables

```
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

#### Directives 
