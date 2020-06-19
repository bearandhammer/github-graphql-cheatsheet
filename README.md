# github-graphql-cheatsheet
Used for jotting down some queries surrounding GraphQL and GitHub.

## Sample queries

Here's a running 'stream of consciousness' based on initially tinkerings with GraphQL and GitHub: 

#### Stock Query (return the login name)

```
query { 
  viewer { 
    login
  }
}
```

#### Get Authenticated User Detail

```
query getAuthenticatedUserDetail {
  viewer {
    id
    name
    login
    url
  }
}
```

### Get Authenticated User Detail Expanded (inc. Status)

```
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
