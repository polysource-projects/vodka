# Vodka

Vodka is a web service to authenticate users with their EPFL account without a Tequila authorization.

## User flow

In this example, let's assume that the user wants to authenticate on `https://adviz.epfl.tools`.

### Step 1: On the ADVIZ website (unauthenticated)

* User opens `https://adviz.epfl.tools`, a website that requires EPFL authentication.
* User clicks on `Login with EPFL account`.
* User gets redirected to `'https://vodka.epfl.tools?redirect_url=' + urlencode('https://adviz.epfl.tools/vodka_callback`).

### Step 2: On Vodka's website

#### Auth-step 1: Checks for a valid JWT

* Vodka checks whether there is or not a valid JWT stored in the cookies. (in this case, auth-step 2 is skipped)

#### Auth-step 2: Gets a new JWT

* Vodka asks for the user's EPFL email address (e.g. `john.doe@epfl.ch`).
* Vodka sends a request to `https://vodka.epfl.tools/api/auth?email=john.doe@epfl.ch`.
* Depending on the response, Vodka shows an error message to the user (e.g. `User not found`) or redirects the user to the code confirmation page.
* Vodka sends a request to `https://vodka.epfl.tools/api/auth/validate?code=985718&email=john.doe@epfl.ch`.
* Depending on the response, Vodka shows an error message to the user (e.g. `Invalid code`).

#### Auth-step 3: Redirects to the ADVIZ website

* Vodka fetches the verified domains `https://vodka.epfl.tools/api/verify?domain=adviz.epfl.tools` to get the domain's verified metadata.
* Depending on the domain, Vodka indicates to the user that the domain whether the domain is verified or not.
* Vodka shows two buttons, `Accept` or `Deny`.
* If `Deny` is clicked, the user is redirected to `https://adviz.epfl.tools/vodka_callback?vodka_error=denied`.
* If `Accept` is clicked, the user is redirected to the ADVIZ website.

### Step 3: On the ADVIZ website (authenticated)

* User gets redirected to `https://adviz.epfl.tools/vodka_callback?vodka_token=JWT` (cf [JWT Data](#jwt-data)).
* ADVIZ's frontend verifies the JWT and shows the user's details on their page accordingly from the JWT or another source.
* If ADVIZ needs to perform some backend operations, it can verify the JWT with Vodka's public key (cf [JWT Data](#jwt-data)).
 
## Vodka API

### Verified domains

* `GET /api/verify`: Returns the domain's metadata if verified. Requires the following parameters:
  * `domain`: The domain to verify.

Response:
* `404`: the domain is not verified (no body sent).
* `200`: 

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
