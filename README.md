# github-graphql-cheatsheet
Used for jotting down some queries surrounding GraphQL and GitHub.

## Sample Queries

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

### Get Detail for a specific User (Arguments)

```
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

```
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

```
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

```
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

