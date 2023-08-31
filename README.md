# vodka specs


## Flows
```md

## Authentication flow

-> /login
    ?redirect=REDIRECT_URI

1. User enters EPFL email address, and requests a 6-digits verifiation code.
    [A] - The server determines if a code should be generated
        - Only 1 code can be sent per email address every 30s from the same device
        - Only 6 emails can be sent from a given IP over 1h
    = 201; The server sets a random QUESTION_ID cookie, or overrides the previous one.
    Code is generated and sent, and `hash(code + QUESTION_ID)` is stored, with generation date.
2. User checks email inbox, and sends a code attempt.
    [B] - The server parses attempt, along with QUESTION_ID. Determines if the attempt can 
            be read:
        - Only 5 attempts are permitted for a given QUESTION_ID.
    Else, if hash matches a record, delete it and send authentication cookie (JWT valid for 
        7 days).
3. User is redirected wherever is needed.
4. Later on, User may log out.
    [C] - The server then revokes existing session IDs, through one of the following methods:
        a. The server stored every valid token hash -> removes them (this solution may be 
            best for handpicking sessions to log out from, later on)
        b. Set a timestamp for each user, and only consider tokens where generation time is 
            greater.


## Authorization flow

-> /authorize
    ?redirect=REDIRECT_URI


> Vodka checks if already authenticated in 1. If not, redirection to Authentication Flow


1. Client fetches current session and stores it in state + through same request, checks if 
    REDIRECT_URI is owned by a verified website.
    [D] - Server fetches from people.epfl.ch various data, based on EPFL email retrieved 
        from session ID cookie.
2. User chooses to either accept or deny to state their identity to website.
3. Client redirects User to REDIRECT_URI
    a. If accepted: 
        -> REDIRECT_URI
            ?token=DATA_JWT
    b. If denied:
        -> REDIRECT_URI
            ?error=denied


```

## API Interactions 

See vodka-server README

## Types

### `User`

```json
{
    "email": string,
    "firstName": string?,
    "lastName": string?
}
```

### `Website`

```json
{
    "domain": "xyz.abc.com",
    "verified": boolean,
    "verifiedSince": number,
    "name": "Abc Xyz",
    "description": "Abc blah blahblah."
}
```
