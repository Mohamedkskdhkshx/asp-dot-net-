# JSON Web Token (JWT)

## 1. What is JWT?

**JWT** stands for **JSON Web Token**. It is a compact format used to
securely transmit information between parties as a JSON object.

### Why do we send a token?

HTTP is a **stateless protocol**. This means that each HTTP request is
independent: by itself, the server does not automatically remember
previous requests or the user's authentication state.

After a user logs in successfully, the server can generate a token and
return it to the client. The client sends that token with later
requests, usually in the HTTP `Authorization` header:

``` http
Authorization: Bearer <your-token>
```

The server validates the token to determine whether the request is
authenticated.

> Important: JWT is one way to implement stateless authentication. The
> server still needs to validate the token on each protected request.

------------------------------------------------------------------------

## 2. The Three Parts of a JWT

A typical signed JWT consists of three parts:

``` text
Header.Payload.Signature
```

The three parts are separated by dots (`.`).

### 2.1 Header

The **Header** contains metadata about the token, including the signing
algorithm and the token type.

Example:

``` json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

-   `alg`: The algorithm used to create the token's signature.
-   `typ`: The type of token. For a JSON Web Token, its value is usually
    `JWT`.

#### What is HS256?

**HS256** stands for **HMAC using SHA-256**.

It is a symmetric signing algorithm. The server uses a secret key to
create the signature, and it uses the same secret key to verify it.

-   **HMAC**: A keyed-hash message authentication code.
-   **SHA-256**: A cryptographic hash function that produces a 256-bit
    hash value.

### 2.2 Payload

The **Payload** contains claims: pieces of information about the user or
the token itself.

Example:

``` json
{
  "sub": "101",
  "name": "Mohammed",
  "email": "mohammed@example.com",
  "role": "Customer",
  "exp": 1790000000
}
```

Some common claims:

  Claim     Meaning
  --------- ------------------------------------------------
  `sub`     Subject of the token, commonly the user's ID
  `name`    User's name (a custom claim in this example)
  `email`   User's email (a custom claim in this example)
  `role`    User's role (a custom claim in this example)
  `exp`     Expiration time, expressed as a Unix timestamp

**Security note:** The payload is Base64URL-encoded, not encrypted.
Anyone who obtains the token can decode and read its header and payload.
Do not put passwords, secret keys, or other sensitive information in the
payload.

### 2.3 Signature

The **Signature** is used to verify that the token has not been changed
and that it was signed by someone who knows the trusted signing key.

For HS256, the signature is calculated conceptually as:

``` text
Signature = HMACSHA256(
    Base64Url(Header) + "." + Base64Url(Payload),
    SecretKey
)
```

The secret key is **used to generate the signature**. It is not placed
inside the JWT and must never be sent to the client.

------------------------------------------------------------------------

## 3. How Does the Server Verify the Signature?

When the client sends a JWT to a protected endpoint, the server performs
validation. Conceptually:

1.  Reads the token's Header and Payload.
2.  Uses the trusted secret key and the specified, permitted algorithm
    to calculate the expected signature.
3.  Compares the expected signature with the signature received in the
    token.
4.  Rejects the token if the signatures do not match.
5.  Checks other validation requirements, such as:
    -   Expiration (`exp`)
    -   Issuer (`iss`), when configured
    -   Audience (`aud`), when configured
    -   Other applicable validation rules

If validation succeeds, the server can accept the token as valid
authentication information and apply authorization rules, such as
checking the user's role.

> A valid signature does not automatically mean that a user is allowed
> to access every endpoint. Authentication establishes who the user is;
> authorization determines what the user is allowed to do.

------------------------------------------------------------------------

## 4. How Is a JWT Encoded?

JWT converts the Header and Payload JSON into **Base64URL-encoded**
strings. It then adds the signature as the third part.

The final structure looks like this:

``` text
Base64Url(Header).Base64Url(Payload).Base64Url(Signature)
```

Base64URL is an encoding format, not encryption. It makes the data
suitable for use in a URL-safe token format; it does not hide the
payload.

------------------------------------------------------------------------

## 5. JWT Implementation Steps in ASP.NET Core

A typical implementation can be organized into these steps:

1.  **Install the required NuGet packages.**
2.  **Configure JWT settings** in `appsettings.json`.
3.  **Configure authentication and JWT Bearer validation** in
    `Program.cs`.
4.  **Create a token generation service** that creates and signs tokens.
5.  **Implement the Login endpoint** to validate credentials and return
    a token.
6.  **Protect endpoints using `[Authorize]`.**

Example of a protected endpoint:

``` csharp
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    return Ok("You are authenticated.");
}
```

The client must send a valid bearer token when calling this endpoint.

------------------------------------------------------------------------

## 6. Key Points to Remember

-   HTTP is stateless; a token lets the client send authentication
    information with each request.
-   A JWT has three parts: **Header**, **Payload**, and **Signature**.
-   The parts are separated by dots.
-   HS256 uses HMAC with SHA-256 and a shared secret key.
-   The secret key is used to create and verify the signature; it is
    never included in the token.
-   The payload is encoded, not encrypted. Do not store secrets in it.
-   The signature helps detect tampering.
-   Validate expiration, issuer, audience, signature, and the other
    validation settings your application requires.
-   `[Authorize]` protects endpoints, while roles and policies can be
    used to control access.
