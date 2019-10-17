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
