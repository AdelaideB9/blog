---
layout: post
title: Heim - UMassCTF '21
date: 2021-03-29T01:30:00
author: samiko
categories:
- ctf
- write-ups
header_img: ''
description: 'Only those who BEARER a token may enter! A web exploitation category challenge on intercepting and forging JSON Web Token from a debugging endpoint to bypass authentication.'

---
## The Heim

Upon navigating to the given URL, we're met with a login form which asks the user for a "name", claiming that "only those who BEARER a token may enter".

![https://i.imgur.com/96m81yH.png](https://i.imgur.com/96m81yH.png)

After entering a name and hitting "Enter", we are then redirected to the `/auth/authorised` page containing our access token:

![https://i.imgur.com/s2PtrcI.png](https://i.imgur.com/s2PtrcI.png)

This likely suggests that we're dealing with some type of bearer token authentication. Bearer tokens allow requests to authenticate by using a cryptic string generated and encrypted by the server, such as a JSON Web Token, which looks something akin to this:

![https://i.imgur.com/IVE3srk.png](https://i.imgur.com/IVE3srk.png)

This token is then included in the HTTP header, in the format of:

```html
Authorization: Bearer <JWT>
```

## Intercepting requests with Burp Suite

Let's intercept the outbound POST request made to `/auth` with Burp Suite's proxy feature, and forward the request to the repeater with `Ctrl-R` to try and figure out what is happening behind the scenes:

![https://i.imgur.com/EZ3LsiC.png](https://i.imgur.com/EZ3LsiC.png)

We see that the browser first makes a POST request to `/auth` with the form data, and is then redirected to the URL:

`/auth?access_token=<JWT>&jwt_secret_key=arottenbranchwillbefoundineverytree`

Leaked at the end of the redirect URL is the `jwt_secret_key`, which is used for encrypting JSON Web Tokens:

`arottenbranchwillbefoundineverytree`

While looking for more API endpoints on the website by trying related keywords, we stumbled upon `/heim`, which returned:

```json
{
  "msg": "Missing Authorization Header"
}
```

This means that the web server is expecting a Bearer token in the HTTP header, so let's add that to our request in Burp Suite's repeater:

![https://i.imgur.com/HR6kSKQ.png](https://i.imgur.com/HR6kSKQ.png)

We received a massive message encoded in base64, let's decode it:

`$ echo "ewogICAgIm...AgIH0KfQ==" | base64 -d`

```json
{
  "api": {
      "v1": {
          "/auth": {
              "get": {
                  "summary": "Debugging method for authorization post",
                  "security": "None",
                  "parameters": {
                      "access_token": {
                          "required": true,
                          "description": "Access token from recently authorized Viking",
                          "in": "path",
                      },
                      "jwt_secret_key": {
                          "required": false,
                          "description": "Debugging - should be removed in prod Heim",
                          "in": "path"
                      }
                  }
              },
              "post": {
                  "summary": "Authorize yourself as a Viking",
                  "security": "None",
                  "parameters": {
                      "username": {
                          "required": true,
                          "description": "Your Viking name",
                          "in": "body",
                          "content": "multipart/x-www-form-urlencoded"
                      }
                  }
              }
          },
          "/heim": {
              "get": {
                  "summary": "List the endpoints available to named Vikings",
                  "security": "BearerAuth"
              }
          },
          "/flag": {
              "get": {
                  "summary": "Retrieve the flag",
                  "security": "BearerAuth"
              }
          }
      }
  }
}
```

Nice! We have obtained a list of all available endpoints and now know the flag is located at `/flag`, and that the GET method for `/auth` was originally meant to be a debugging endpoint, explaining why `jwt_secret_key` was leaked in the redirect URL.

Making a GET request to `/flag` with our Bearer token returns:

```json
{
  "msg": "You are not worthy. Only the AllFather Odin may view the flag"
}
```

Seems like only Odin is worthy enough to view the flag! Also, trying to submit "odin" as the name in the login form returns:

```json
{
  "error": "You are not wise enough to be Odin"
}
```

Though, since we can obtain a sample access token and are in possession of the secret key, we can forge our own tokens to authenticate as the Odin user.

## Forging Odin's token

Using [CyberChef](https://gchq.github.io/CyberChef/) tool, we can easily read the payload of our own JWT string with the "JWT decode" function:

![https://i.imgur.com/gRvcmbl.png](https://i.imgur.com/gRvcmbl.png)

We see that the JWT payload contains the following data:

```json
{
    "fresh": false,
    "iat": 1617017482,
    "jti": "380b9f5c-fb31-479e-a189-59e2c8040453",
    "nbf": 1617017482,
    "type": "access",
    "sub": "samiko",
    "exp": 1617018382
}
```

Of all variables, `sub` (subject) and `exp` (expiration time) are the ones that appear most interesting to us, since we want to forge a token for Odin (that never expires!), so let's modify the payload to the following:

```json
{
    "fresh": false,
    "iat": 1617017482,
    "jti": "380b9f5c-fb31-479e-a189-59e2c8040453",
    "nbf": 1617017482,
    "type": "access",
    "sub": "odin",
    "exp": 9999999999
}
```

Using the "JWT Sign" function with the leaked `jwt_secret_key` and HS256 as parameters, we get the token:

![https://i.imgur.com/AEjoHH4.png](https://i.imgur.com/AEjoHH4.png)

Using the forged token in the HTTP header as a Bearer token, we make a GET request to `/flag`:

![https://i.imgur.com/Tq2jaX7.png](https://i.imgur.com/Tq2jaX7.png)

Voila! We got the flag:

```json
UMASS{liveheim_laughheim_loveheim}
```

## Resources

1. [https://swagger.io/docs/specification/authentication/bearer-authentication/](https://swagger.io/docs/specification/authentication/bearer-authentication/)
2. [https://research.securitum.com/jwt-json-web-token-security/](https://research.securitum.com/jwt-json-web-token-security/)
3. [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)