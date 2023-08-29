# Vodka

Vodka is a way to authenticate users with their EPFL account without a Tequila authorization.

## User flow

In this example, let's assume that the user wants to authenticate on `https://adviz.epfl.tools`.

### Step 1: On the ADVIZ website (unauthenticated)

* User opens `https://adviz.epfl.tools`, a website that requires EPFL authentication.
* User clicks on `Login with EPFL account`.
* User gets redirected to `https://vodka.epfl.tools?redirect_url=https://adviz.epfl.tools`.

### Step 2: On Vodka's website

* Vodka fetches the verified domains `https://vodka.epfl.tools/verified` (response e.g. `[adviz.epfl.tools, flep.ch]`)
* Depending on the domain, Vodka shows a notice to the user that the domain is not verified during the complete flow.
* Vodka asks for the user's EPFL email address (e.g. `john.doe@epfl.ch`).
* Vodka sends a request to `https://vodka.epfl.tools/api/auth?email=john.doe@epfl.ch`.
* Depending on the response, Vodka shows an error message to the user (e.g. `User not found`) or redirects the user to the code confirmation page.
* Vodka sends a request to `https://vodka.epfl.tools/api/auth/validate?code=985718&email=john.doe@epfl.ch`.
* Depending on the response, Vodka shows an error message to the user (e.g. `Invalid code`) or redirects the user to the ADVIZ website.

### Step 3: On the ADVIZ website (authenticated)

* User gets redirected to `https://adviz.epfl.tools?token=JWT_TOKEN` (cf [JWT Data](#jwt-data)).
* ADVIZ's frontend verifies the JWT token and shows the user's details on their page accordingly.
* If ADVIZ needs to perform some backend operations, it can verify the JWT token with Vodka's public key (cf [JWT Data](#jwt-data)).
 
## Vodka API

### Verified domains

* `GET /verified`: Returns a list of verified domains (e.g. `[adviz.epfl.tools, flep.ch]`).

### Start authentication

* `POST /api/auth`: Starts an authentication with the user. Requires the following parameters:
  * `email`: The user's EPFL email address.

Response:

* `404`: the user can not be found on `people.epfl.ch` (no body sent).
* `429`: the server can not handle the request (no body sent).
* `200`: the user has been found on `people.epfl.ch` and an email has been sent to the user's EPFL email address. Response body:
```json
{
    "status": "code_mail_sent",
    "profile_data": {
        "firstName": "John",
        "lastName": "Doe",
        "type": "student",
        "course": "GM-BA5",
        "email": "john.doe@epfl.ch"
    }
}
```

**Important**: calling this endpoint always cancel the previous authentication request. It means that if the user has already requested an authentication code, the previous code will be invalidated.

### Validate an authentication code

* `POST /api/auth/validate`: Validates an authentication code. Requires the following parameters:
  * `email`: The user's EPFL email address.
  * `code`: The authentication code sent by email.

Response:

* `404`: the user can not be found on `people.epfl.ch` (no body sent).
* `429`: the server can not handle the request (no body sent).
* `403`: the code is invalid (no body sent).
* `200`: the code is valid. Response body:
```json
{
    "status": "code_validated",
    "profile_data": {
        "firstName": "John",
        "lastName": "Doe",
        "type": "student",
        "course": "GM-BA5",
        "email": "john.doe@epfl.ch"
    },
    "jwt": "..."
}
```

#### JWT Data

**Note**: the JWT is signed by the Vodka server and the public key can be fetched at `https://vodka.epfl.tools/jwt_public_key`.

```json
{
    "sub": "john.doe@epfl.ch",
    "iss": "https://vodka.epfl.tools",
    "aud": "https://adviz.epfl.tools",
    "exp": 1580515200,
    "iat": 1580511600,
    "profile_data": {
        "firstName": "John",
        "lastName": "Doe",
        "type": "student",
        "course": "GM-BA5",
        "email": "john.doe@epfl.ch"
    }
}
```

