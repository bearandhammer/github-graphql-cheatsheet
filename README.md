# github-graphql-cheatsheet
Used for jotting down some queries surrounding GraphQL and GitHub.

## Sample Queries

Here's a running 'stream of consciousness' based on initially tinkerings with GraphQL and GitHub: 

#### Stock Query (return the login name)

```graphql
query { 
  viewer { 
    login
  }
}
```

#### Get Authenticated User Detail

```graphql
query getAuthenticatedUserDetail {
  viewer {
    id
    name
    login
    url
  }
}
```

#### Get Authenticated User Detail Expanded (inc. Status)

```graphql
query getAuthenticatedUserDetailWithStatus {
  viewer {
    id
    name
    login
    url
    location
    bio
    status {
      emoji
      message
    }
  }
}
```

#### Get Detail for a specific User (Arguments)

```graphql
{
  user(login:"craigvincent") {
    id
    name
    url
    bio
  }
}
```

#### Get Details for multiple Users (Alias)

```graphql
{
  craig:user(login:"craigvincent") {
    id
    name
    url
    bio
  }
  bear:user(login:"bearandhammer") {
    id
    name
    url
    bio
  }
}
```

#### Alias applied to a particular Field

```graphql
{
  craig:user(login:"craigvincent") {
    id
    name
    url
    bio
  }
  bear:user(login:"bearandhammer") {
    user_id:id
    name
    url
    bio
  }
}
```

#### Simple Fragment Usage

Here we want a simple way to define the User-based fields we want to return (without needing to make adjustments throughout several areas of the code):

```graphql
{
  craig:user(login:"craigvincent") {
    ...userFields
  }
  bear:user(login:"bearandhammer") {
    ...userFields
  }
}

fragment userFields on User {
    id
    name
    url
    bio
}
```

#### Apply an Alias to a Fragment

```graphql
{
  craig:user(login:"craigvincent") {
    ...userFields
  }
  bear:user(login:"bearandhammer") {
    ...userFields
  }
}

fragment userFields on User {
    user_id:id
    name
    url
    bio
}
```

#### Specific change on top of a Fragment

```graphql
{
  craig:user(login:"craigvincent") {
    ...userFields
  }
  bear:user(login:"bearandhammer") {
    ...userFields
    location
  }
}

fragment userFields on User {
    user_id:id
    name
    url
    bio
}
```

#### Inline Fragment

```graphql
{
  user(login:"bearandhammer") {
    ... on Actor {
      login
      url
    }
    ... on ProfileOwner {
    	id
      name
    }
    ... on UniformResourceLocatable {
      res_loc:url
      resourcePath
    }
    bio
  }
}
```

#### Variable Usage

Here a named query is created specifying that a variable named `$loginName` of type string should be provided. The exclamation mark denotes that this variable is mandatory.

```graphql
query ($loginName: String!) {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
  }
}
```

##### Query Variables Definition

```graphql
{
  "loginName": "bearandhammer"
}
```

#### Variable Usage (optional)

In this instance trhe `$loginName` variable is not mandatory. A default value of `craigvincent` is supplied:

```graphql
query ($loginName: String = "craigvincent") {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
  }
}
```

#### Inline Variable (inside Fragment)

```graphql
query ($loginName: String!, $repoName: String!) {
  user(login: $loginName) {
    ...userFields
  }
}

fragment userFields on User {
  id
  name
  url
  bio,
  repository(name: $repoName) {
    name
    url
  }
}
```

##### Query Variables Definition

```graphql
{
  "loginName": "bearandhammer",
  "repoName": "orchard-core-sample"
}
```

#### Directives

NOTES:

- Changes to the query structure.
- Attach to Field or Fragments.
- `@Include` & `@Skip`.

@DirectiveName (if: Boolean).

#### Include and Skip

Here we include/skip the `bio`/`status` properties based on the `$loadStatus` variable:

```graphql
query ($loadStatus: Boolean!) {
  viewer {
    id
    name
    login
    url
    location
    bio @skip(if: $loadStatus)
    status @include(if: $loadStatus) {
      emoji
      message
    }
  }
}
```

##### Query Variables Definition

```graphql
{
  "loadStatus": true
}
```

#### Pagination

- `repositories` - consumes a value (we want to pull the first 3 values through).
- `totalCount` - the count of entities in the collection.
- `pageInfo` - allows access to properties denoting if we have a previous/next page. A `Cursor` denotes a unique identifier for a particular record.
- `edges` - allows access to the physical information behind the first three records via `node` (notice how the `cursor` values tie back to the `pageInfo` `startCursor` & `endCursor`, as expected).

```graphql
query ($loginName: String = "bearandhammer") {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
    repositories(first: 3) {
      totalCount
      pageInfo {
        hasPreviousPage
        hasNextPage
        startCursor
        endCursor
      }
      edges {
        cursor
        node {
          name
          url
        }
      }
    }
  }
}
```

##### Example Output

```json
{
  "data": {
    "user": {
      "id": "MDQ6VXNlcjUxMTM1MTgy",
      "name": "Lewis Grint",
      "login": "bearandhammer",
      "url": "https://github.com/bearandhammer",
      "location": "Attleborough, Norfolk",
      "bio": "I'm a software developer who loves tinkering with anything I can get my hands on! Random topics and forays off the beaten track = me.",
      "repositories": {
        "totalCount": 15,
        "pageInfo": {
          "hasPreviousPage": false,
          "hasNextPage": true,
          "startCursor": "Y3Vyc29yOnYyOpHOC0VLtw==",
          "endCursor": "Y3Vyc29yOnYyOpHOD9mZUw=="
        },
        "edges": [
          {
            "cursor": "Y3Vyc29yOnYyOpHOC0VLtw==",
            "node": {
              "name": "portfolio",
              "url": "https://github.com/bearandhammer/portfolio"
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpHOD8dKMQ==",
            "node": {
              "name": "typescript-forays",
              "url": "https://github.com/bearandhammer/typescript-forays"
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpHOD9mZUw==",
            "node": {
              "name": "github-actions-web-app",
              "url": "https://github.com/bearandhammer/github-actions-web-app"
            }
          }
        ]
      }
    }
  }
}
```

#### Pagination (next page)

Here, the `after` property value is set to the `endCursor` value, that gracefully moves us on to the next page of records:

```graphql
query ($loginName: String = "bearandhammer") {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
    repositories(first: 3, after: "Y3Vyc29yOnYyOpHOD9mZUw==") {
      totalCount
      pageInfo {
        hasPreviousPage
        hasNextPage
        startCursor
        endCursor
      }
      edges {
        cursor
        node {
          name
          url
        }
      }
    }
  }
}
```

##### Example Output

```json
{
  "data": {
    "user": {
      "id": "MDQ6VXNlcjUxMTM1MTgy",
      "name": "Lewis Grint",
      "login": "bearandhammer",
      "url": "https://github.com/bearandhammer",
      "location": "Attleborough, Norfolk",
      "bio": "I'm a software developer who loves tinkering with anything I can get my hands on! Random topics and forays off the beaten track = me.",
      "repositories": {
        "totalCount": 15,
        "pageInfo": {
          "hasPreviousPage": true,
          "hasNextPage": true,
          "startCursor": "Y3Vyc29yOnYyOpHOD91wvA==",
          "endCursor": "Y3Vyc29yOnYyOpHOD_QlMg=="
        },
        "edges": [
          {
            "cursor": "Y3Vyc29yOnYyOpHOD91wvA==",
            "node": {
              "name": "netcore-starter-web-app-test",
              "url": "https://github.com/bearandhammer/netcore-starter-web-app-test"
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpHOD-a2pA==",
            "node": {
              "name": "bearandhammer.github.io",
              "url": "https://github.com/bearandhammer/bearandhammer.github.io"
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpHOD_QlMg==",
            "node": {
              "name": "windows-terminal-theme",
              "url": "https://github.com/bearandhammer/windows-terminal-theme"
            }
          }
        ]
      }
    }
  }
}
```

#### Pagination (last records)

It is possible to retrieve the last 'x' records using `last`, as follows:

```graphql
query ($loginName: String = "bearandhammer") {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
    repositories(last: 3) {
      totalCount
      pageInfo {
        hasPreviousPage
        hasNextPage
        startCursor
        endCursor
      }
      edges {
        cursor
        node {
          name
          url
        }
      }
    }
  }
}
```

From here, you can specify the `before` argument using the `startCursor` value:

```graphql
query ($loginName: String = "bearandhammer") {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
    repositories(last: 3, before: "Y3Vyc29yOnYyOpHOED3_XQ==") {
      totalCount
      pageInfo {
        hasPreviousPage
        hasNextPage
        startCursor
        endCursor
      }
      edges {
        cursor
        node {
          name
          url
        }
      }
    }
  }
}
```

#### Sorting

Use the `orderBy` argument as illustrated (to select a target field and sort direction):

```graphql
query ($loginName: String = "bearandhammer") {
  user(login: $loginName) {
    id
    name
    login
    url
    location
    bio
    repositories(last: 3, before: "Y3Vyc29yOnYyOpHOED3_XQ==", orderBy: {
      field: NAME
      direction: DESC
    }) {
      totalCount
      pageInfo {
        hasPreviousPage
        hasNextPage
        startCursor
        endCursor
      }
      edges {
        cursor
        node {
          name
          url
        }
      }
    }
  }
}
```

## GraphQL Schema

Basic set - defines the supported:
- Queries
- Mutations
- Fields
- Types

Schema - used to determine if a query is valid (GraphQL Schema Language).

#### Types

- Basic components.
- Sent/received by the server.
- Scalar Type.
- Object Type.

Scalar Type, Object Type, Interface Type, Union Type, Enum Type, Input Object Type.

#### Scalar Type

Represents a primitive value (Integer, Float, String, Boolean and ID). We can  opt to add/omit types as required (use the `scalar` keyword).

```graphql
scalar Date
```

#### Object Type

Describes a more complex entities, consists of fields (which can be Scalar or Object types).

```graphql
type User {
  id: ID
  userName: String
  mailId: String
  contactDetails: ContactDetails
}

type ContactDetails {
  street: String
  city: String
  state: String
  zipCode: String
}
```

#### Interface

Much like any other language, an Interface is an abstract type that includes a certain set of fields that a type must include to implement the interface. Multiple interfaces can be implemented also:

```graphql
interface BaseUser {
  userName: String
  password: String
}

interface BaseAdmin {
  canGenerateReport: Boolean
  canManageUsers: Boolean
}

type Admin implements BaseUser & BaseAdmin {
  userName: String
  password: String
  canGenerateReport: Boolean
  canManageUsers: Boolean
}
```

#### Union Type

Union types are very similar to interfaces, but they don't get to specify any common fields between the types.

Example:

```graphql
type WebResult {
  webPageUrl: Url
  description: String
}

type PhotoResult {
  imageUrl: Url
  height: Int
  width: Int
}

union SearchResult = WebResult | PhotoResult

type SearchQuery {
  searchResult: SearchResult
}
```

##### Execute Query

```graphql
query {
  searchResult {
    ... on WebResult {
      webPageUrl
    }
    ... on PhotoResult {
      imageUrl
    }
  }
}
```

#### Enum Type

Enumeration types are a special kind of scalar that is restricted to a particular set of allowed values (again, much like other languages) - note the naming convention for the allowed values:

```graphql
enum season {
  SPRING
  SUMMER
  AUTUMN
  WINTER
}
```

#### InputObject Type

If you need complex argument data then this is the ticket. This type is particularly valuable in the case of mutations, where you might want to pass in a whole object to be created. The Fields can be Scalar, Enum or of InputObject type.

```graphql
input AddressDetail {
  street: String
  city: String
  state: String
  zipCode: String
}
```

#### List

We can use a type modifier to mark a type as a List, which indicates that this field will return an array of that type. In the schema language, this is denoted by wrapping the type in square brackets. Can be used on Scalar and Object types.

```graphql
type TravelDetails {
  countriesVisited: [CountryEnumType]
  visitDetails: [VisitDetailsObjectType]
}
```

### Non-Null

Fields are nullable by default, but the exclamation mark can be used to explicitly state a value is not nullable. This can be used against List types also. Can be used on Scalar and Object types. Basic example:

```graphql
type OrderDetails {
  userMail: String!
  cartItems: [CartItem]!
}
```
