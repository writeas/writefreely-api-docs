---
title: Write.as API Documentation

language_tabs:
  - shell
  - go

toc_footers:
  - <a href='https://github.com/tripit/slate'>Docs by Slate</a>

search: true
---

# Introduction

Welcome to the [Write.as](https://write.as) API! Our API gives you full access to Write.as data and lets you build your own applications or utilities on top of it.

Our API is accessible at **https://write.as/api/** (HTTPS _only_) and via our Tor hidden service at **writeas7pm7rcdqg.onion/api/**.

Backwards compatibility is important to us since we have a large set of [clients](https://write.as/apps) in the wild. As we add new features, we usually add endpoints and properties alongside existing ones, but don't remove or change finalized ones that are documented here. If a breaking API change is required in the future, we'll version new endpoints and update these docs.

This documentation represents the officially-supported, latest API. Any properties or endpoints you discover in the API that aren't documented here should be considered experimental or beta functionality, and subject to change without any notice.

## Errors

> Failed requests return with an error `code` and `error_msg`, like below.

```json
{
  "code": "400",
  "error_msg": "Bad request."
}
```

The Write.as API uses conventional HTTP response codes to indicate the status of an API request. Codes in the `2xx` range indicate success or that more information is available in the returned data, in the case of bulk actions.
Codes in the `4xx` range indicate that the request failed with the information provided. Codes in the `5xx` range indicate an error with the Write.as servers (you shouldn't see many of these).

Error Code | Meaning
---------- | -------
400 | Bad Request -- The request didn't provide the correct parameters, or JSON / form data was improperly formatted.
401 | Unauthorized -- No valid user token was given.
403 | Forbidden -- You attempted an action that you're not allowed to perform.
404 | Not Found -- The requested resource doesn't exist.
405 | Method Not Allowed -- The attempted method isn't supported.
410 | Gone -- The entity was unpublished, but may be back.
429 | Too Many Requests -- You're making too many requests, especially to the same resource.
500, 502, 503 | Server errors -- Something went wrong on our end.

## Responses

> Successful requests return with a `code` in the `2xx` range and a `data` object or array, depending on the request.

```json
{
  "code": "200",
  "data": {
  }
}
```

All responses are returned in a JSON object containing two fields:

Field | Type | Description
----- | ---- | -----------
`code` | integer | An HTTP status code in the `2xx` range.
`data` | object or array | A single object for most requests; an array for bulk requests.

This wrapper will never contain an `error_msg` property at the top level.

<aside class="success">
Most endpoints <em>accept</em> both form data or JSON, assuming form data by default unless a <code>Content-Type: application/json</code> header is sent.
</aside>


# Authentication

The API doesn't require any authentication, either for the client or end user. However, if you want to perform actions on behalf of a user, you'll need to pass a user access token with any requests:

`Authorization: 00000000-0000-0000-0000-000000000000`

<aside class="notice">
Replace <code>00000000-0000-0000-0000-000000000000</code> with a user's access token.
</aside>

## Authenticate a User

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/auth/login" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"alias": "matt", "pass": "12345"}'
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "access_token": "00000000-0000-0000-0000-000000000000",
    "user": {
      "username": "matt",
      "email": "matt@example.com",
      "created": "2015-02-03T02:41:19Z"
    }
  }
}
```

Authenticates a user with Write.as, creating an access token for future authenticated requests.

Users can only authenticate with their primary account, i.e. the first collection/blog they created, which may or may not have multiple collections associated with it.

### Definition

`POST https://write.as/api/auth/login`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**alias** | string | yes | The user's username / alias.
**pass** | string | yes | The user's password.

### Returns

A user access token and the authenticated user's full data.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 401,
  "error_msg": "Incorrect password."
}
```

Error Code | Meaning
---------- | -------
400 | Request is missing required information, or bad form data / JSON.
401 | Incorrect information given.
404 | User doesn't exist.
429 | You're trying to log in too many times too quickly. You shouldn't see this unless you're doing something bad (in which case, please stop that).

# Posts

Posts are the most basic entities on Write.as. Each can have its own appearance, and isn't connected to any other Write.as entity by default. They can exist without an owner (user), or with an owner but no collection, or with both an owner and collection.

When posts are published with no owner, it's up to the client to remember the posts a user has published. By keeping this information on the client, users can write anonymously -- both according to Write.as and to anyone who reads any given post.

Users can choose between different appearances for each post, usually passed to and from the API as a `font` property:

| Argument | Appearance (Typeface) | Word Wrap? | Also on |
| -------- | --------------------- | ---------- | ------- |
| `sans` | Sans-serif (Open Sans) | Yes | _N/A_ |
| `serif` / `norm` | Serif (Lora) | Yes | _N/A_ |
| `wrap` | Monospace | Yes | _N/A_ |
| `mono` | Monospace | No | paste.as |
| `code` | Syntax-highlighted monospace | No | paste.as |

## Publish a Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/posts" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"body": "This is a post.", "title": "My First Post"}'
```

> Example Response

```json
{
  "code": 201,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M",
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:46Z",
    "title": "My First Post",
    "body": "This is a post.",
    "tags": [
    ]
  }
}
```

This creates a new post, associating it with a user account if authenticated. If the request is successful, the post will be available at:

* write.as/`{id}`
* writeas7pm7rcdqg.onion/`{id}`
* paste.as/`{id}` -- if `font` was _code_ or _mono_

### Definition

`POST https://write.as/api/posts`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | The content of the post.
**title** | string | no | An optional title for the post. If none is given, Write.as will try to extract one for the browser's title bar, but not change any appearance in the post itself.
**font** | string | no | One of any post appearance types [listed above](#posts). If invalid, defaults to `serif`.
**lang** | string | no | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | no | Whether or not the content should be display right-to-left, for example if written in Arabic or Hebrew.
**crosspost** | array | no | **Must be authenticated**. An array of integrations and associated usernames to post to. See [Crossposting](#crossposting).

### Returns

The newly created post.

<aside class="notice">
When the request is unauthenticated, the client should store the <code>token</code> returned from this request so users can modify their post later. Otherwise it isn't necessary.
</aside>

## Retrieve a Post

```go
s := post.Get("rf3t35fkax0aw")
```

```shell
curl https://write.as/api/posts/rf3t35fkax0aw
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:05Z",
    "title": "My First Post",
    "body": "This is a post.",
    "tags": [  
    ],
    "views": 0
  }
}
```

This retrieves a post entity. It includes extra Write.as data, such as page views and extracted hashtags.

### Definition

`GET https://write.as/api/posts/{POST_ID}`


## Update a Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/posts/rf3t35fkax0aw" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M", "body": "My post is updated."}'
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:05Z",
    "title": "My First Post",
    "body": "This is an updated post.",
    "tags": [  
    ],
    "views": 0
  }
}
```

This updates an existing post.

### Definition

`POST https://write.as/api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | The content of the post.
**title** | string | no | An optional title for the post. Supplying a parameter but leaving it blank will remove any title currently on the post.
**font** | string | no | One of any post appearance types [listed above](#posts). If invalid, it doesn't change.
**lang** | string | no | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | no | Whether or not the content should be display right-to-left, for example if written in Arabic.

### Returns

The entire post as it stands after the update.


## Unpublish a Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/posts/rf3t35fkax0aw" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M", "body": ""}'
```

> Response

```json
{
  "code": 410,
  "error_msg": "Post unpublished by author."
}
```

<aside class="warning">
<strong>Unfinalized design</strong>.
</aside>

This unpublishes an existing post. Simply pass an empty `body` to an existing post you're authorized to update, and in the future that post will return a `410 Gone` status.

The client will likely want to maintain what was in the body before the post was unpublished so it can be restored later.

### Definition

`POST https://write.as/api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | Should be an empty string (`""`).

### Returns

An error `410` with a message: _Post unpublished by author._


## Delete a Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/posts/rf3t35fkax0aw?token=ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M" \
  -X DELETE
```

> A successful deletion returns a `200` with no content in the body.

This deletes a post.

### Definition

`DELETE https://write.as/api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**token** | string | _if unauth'd_ | The post's modify token.

### Returns

A `200` status code and no content in the body.

<aside class="warning">
<strong>Upcoming breaking change</strong>: Soon this will return a <code>204</code>. Current clients should handle both status codes as indicating a successful operation.
</aside>

# Collections

Collections are referred to as blogs on most of Write.as. They're groups of posts that currently can only be _public_, i.e. available at a publicly-accessible URL.

Each user has one collection matching their username, but can also have more collections connected to their account as a Pro user.

<aside class="notice">
All collection requests except retrieval must be <a href="#authentication">authenticated</a>.
</aside>

## Create a Collection

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/collections" \
  -H "Authorization: 00000000-0000-0000-0000-000000000000" \
  -X POST \
  -d '{"alias": "new-blog", "title": "The Best Blog Ever"}'
```

> Example Response

```json
{
  "code": 201,
  "data": {
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "",
    "style_sheet": "",
    "email": "new-blog-wjn6epspzjqankz41mlfvz@writeas.com",
    "total_posts": 0
  }
}
```

This creates a new collection. **Pro user required**.

### Definition

`POST https://write.as/api/collections`

### Arguments

If only a `title` is given, the `alias` will be generated on the server from it. Clients should store the returned `alias` for future operations.

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**title** | string | yes | An optional title for the post. If none is given, Write.as will try to extract one for the browser's title bar, but not change any appearance in the post itself.
**alias** | string | no | One of any post appearance types [listed above](#posts). If invalid, defaults to `serif`.

### Returns

The newly created collection.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 403,
  "error_msg": "You must be a Pro user to do that."
}
```

Error Code | Meaning
---------- | -------
400 | Request is missing required information, or bad form data / JSON.
401 | Incorrect information given.
403 | You're not a Pro user.
409 | Alias is taken.
412 | You've reached the maximum number of collections allowed.


## Retrieve a Collection

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl https://write.as/api/collections/new-blog
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "",
    "style_sheet": "",
    "total_posts": 0
  }
}
```

This retrieves a collection and its metadata.

### Definition

`GET https://write.as/api/collections/{COLLECTION_ALIAS}`

### Returns

The requested collection.


## Publish a Collection Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/collections/new-blog/posts" \
  -H "Authorization: 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"body": "This is a blog post.", "title": "My Post"}'
```

> Example Response

```json
{
  "code": 201,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M",
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:46Z",
    "title": "My First Post",
    "body": "This is a post.",
    "tags": [
    ]
  }
}
```

This creates a new post and adds it to the given collection. User must be authenticated.

### Definition

`POST https://write.as/api/collections/{COLLECTION_ALIAS}/posts`

### Arguments

This accepts the same arguments as [anonymous posts](#publish-a-post), plus an optional `crosspost` parameter.

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | The content of the post.
**title** | string | no | An optional title for the post. If none is given, Write.as will try to extract one for the browser's title bar, but not change any appearance in the post itself.
**font** | string | no | One of any post appearance types [listed above](#posts). If invalid, defaults to `serif`.
**lang** | string | no | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | no | Whether or not the content should be display right-to-left, for example if written in Arabic or Hebrew.
**crosspost** | array | no | An array of integrations and associated usernames to post to. See [Crossposting](#crossposting).

### Returns

The newly created post.


## Retrieve Collection Posts

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl https://write.as/api/collections/new-blog/posts
```

> Example Response

```json
{ 
  "code":200,
  "data": {  
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "",
    "style_sheet": "",
    "private": false,
    "total_posts": 1,
    "posts": [
      {
        "id": "hjb7cvwaevy9eayp",
        "slug": "my-post",
        "appearance": "norm",
        "language": "",
        "rtl": false,
        "listed": false,
        "created": "2016-07-09T14:29:33Z",
        "title": "My Post",
        "body": "This is a blog post.",
        "tags": [
        ],
        "views": 0
      }
    ]
  }
}
```

This retrieves a collection's posts along with the collection's data.

### Definition

`GET https://write.as/api/collections/{COLLECTION_ALIAS}/posts`

### Returns

The requested collection and its posts in a `posts` array.


# Users

Currently it's only possible to create users, but the API isn't finalized.

See [Authentication](#authenticate-a-user) for information on logging in.


# Integrations

Write.as integrates with other services on the web to provide features like crossposting upon publishing a Write.as post.

On the web, these are known as **channels**, and if you're logged in, they can be added [here](https://write.as/me/c/).

## Crossposting

> Example `crosspost` Parameter

```json
"crosspost": [
  { "twitter": "writeas__" },
  { "twitter": "ilikebeans" },
  { "medium": "someone" },
  { "tumblr": "example" }
]
```

Crossposting is easy to do when publishing a new post. It requires authentication and at least one connected integration / channel on the web.

<aside class="notice">
Soon you'll be able to retrieve a user's connected accounts, so this may only be relevant to users posting from their own account for now.
</aside>

For the `crosspost` parameter on a new post, pass a JSON array of single-property objects. The object's sole property should be the ID of the integration you want to crosspost to (see below), and its value should be the user's connected username _on that service_.

ID | Integration
--- | ----------
twitter | Twitter
tumblr | Tumblr
medium | Medium


---

Missing something? Questions? Find us on [Slack](http://slack.write.as), [IRC](irc://irc.freenode.net/writeas), or [other places](https://write.as/contact) and ask us.

