---
title: Write.as API Documentation

language_tabs:
  - shell

toc_footers:
  - <a href='https://github.com/tripit/slate'>Docs by Slate</a>

search: true
---

# Introduction

Welcome to the Write.as API! Use this to interact with Write.as.

Write.as works well anonymously or as a registered user. It can also be used via our Tor hidden service at `writeas7pm7rcdqg.onion/api/`.

Most endpoints accept both form data or JSON, assuming form data unless a `Content-Type: application/json` header is sent.

# Errors

The Write.as API uses conventional HTTP response codes to indicate the status of an API request. Codes in the `2xx` range indicate success or that more information is available in the returned data, in the case of bulk actions.
Codes in the `4xx` range indicate that the request failed with the information provided. Codes in the `5xx` range indicate an error with the Write.as servers (you shouldn't see many of these).

Error Code | Meaning
---------- | -------
400 | Bad Request -- The request didn't provide the correct parameters.
401 | Unauthorized -- No valid user token was given.
403 | Forbidden -- You attempted an action that you're not allowed to perform.
404 | Not Found -- The requested resource doesn't exist.
405 | Method Not Allowed -- The attempted method isn't supported.
410 | Gone -- The entity was unpublished, but may be back.
429 | Too Many Requests -- You're making too many requests, especially to the same resource.
500, 502, 503 | Server errors -- Something went wrong on our end.

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

> All endpoints will work with authenticated requests.

Write.as doesn't require authentication to use, either for the client or end user.

However, if you want to perform actions on behalf of a user, you'll need to pass a user access token with any requests:

`Authorization: 00000000-0000-0000-0000-000000000000`

<aside class="notice">
Replace <code>00000000-0000-0000-0000-000000000000</code> with a user's access token.
</aside>

# Posts

Posts are very flexible entities on Write.as. Each can have its own appearance, and isn't connected to any other Write.as entity by default. They can exist without an owner, or with an owner but no collection, or with both an owner and collection.

When posts have no owner, it's up to the client to maintain posts a user has created. By keeping this information on the client, users can write anonymously -- both according to Write.as and to anyone who reads any given post.

Users can choose between different appearances for each post, usually passed to and from the API as a `font` property:

| Argument | Appearance (Typeface) | Word Wrap? | Also on |
| -------- | --------------------- | ---------- | ------- |
| `sans` | Sans-serif (Open Sans) | Yes | _N/A_ |
| `serif` | Serif (Lora) | Yes | _N/A_ |
| `wrap` | Monospace | Yes | _N/A_ |
| `mono` | Monospace | No | paste.as |
| `code` | Syntax-highlighted monospace | No | paste.as |

## Publish a Post

```shell
curl "https://write.as/api/posts" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"body": "This is a basic post.", "title": "My First Post"}'
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
    "body": "This is a basic post.",
    "tags": [
    ]
  }
}
```

This creates a new anonymous post. If successful, the post will be available at:

* write.as/`{id}`
* writeas7pm7rcdqg.onion/`{id}`
* paste.as/`{id}` -- if `font` was _code_ or _mono_

### Definition

`POST https://write.as/api/posts`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | true | The content of the post.
**title** | string | false | An optional title for the post. If none is given, Write.as will try to extract one for the browser's title bar, but not change any appearance in the post itself.
**font** | string | false | One of any post appearance types [listed above](#posts). If invalid, defaults to `serif`.
**lang** | string | false | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | false | Whether or not the content should be display right-to-left, for example if written in Arabic.

### Returns

The newly created post.

<aside class="notice">
When unauthenticated, the client should store the <code>token</code> it gets back from this request so users can modify their post later. Otherwise it isn't necessary.
</aside>

## Retrieve a Post

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
    "body": "This is a basic post.",
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

This updates a post.

### Definition

`POST https://write.as/api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | true | The content of the post.
**title** | string | false | An optional title for the post. Supplying a parameter but leaving it blank will remove any title currently on the post.
**font** | string | false | One of any post appearance types [listed above](#posts). If invalid, it doesn't change.
**lang** | string | false | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | false | Whether or not the content should be display right-to-left, for example if written in Arabic.

### Returns

The entire post as it stands after the update.
