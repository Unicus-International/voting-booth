# Voting Booth API

## User life cycle management

### Registration

Registration accepts as parameters an email address, a name, and two passwords. Registration attempts will be rejected with a status of 403 in the case of an already registered email address, with a status of 400 in the case of an empty name, a malformed email address, or non-matching passwords. In the case of a status code of 400, the reply body will contain a JSON message detailing the specific error, otherwise the reply body will be empty.

```
POST /user/register

request parameters:
  name: string
  email_address: string
  password_one: string
  password_two: string

reply status:
  201: Registration successful
  400: Invalid request
  403: User already exists

reply body:
  none except:
    400: application/json: { "error": <reason> }
```

### Login

Login accepts as parameters an email address and a password. Login attempts will be rejected with a status of 403 if the password does not match the password for the user account, or if the user account does not exist. Otherwise, if the user exists and the password is valid, the returned status will be 204, and the `X-Session-Id` will be set. **All other requests to the API will, unless otherwise noted, require the contents of this header to be set as a request header.** The reply body will be empty.

```
POST /user/login

request parameters:
  email_address: string
  password: string

reply status:
  204: Login successful
  403: Wrong password

reply headers:
  X-Session-Id: <session>

reply body:
  none
```

### Logout

Logout accepts no parameters. The reply will have a status code of 204, and the body of the response will be empty. Calling this method invalidates the contents of `X-Session-Id`, and will cause any other call involving it return an error 403, excepting this method, which acts idempotently.

```
GET /user/logout

request headers:
  X-Session-Id: <session>

reply status:
  204: Logout successful
```

## Voting

Vote accepts no parameters, except the in-URL franchise ID parameter. Voting also does not expect or require the `X-Session-Id` header, and any supplied header of the sort will be ignored. If the franchise does not exist, the reply body will be empty.

### Fetching vote information

The reply body will contain a JSON object that contains a field `election`, containing the fields `name`, a string; `question`, a string; `runs`, a JSON object containing `from` and `to`, both dates (ISO8601-encoded strings); and `system`, string containing an identifier for the system in use.

The reply JSON object will also have the field `ballots` which contains a list of ballots, each a JSON object with a field `name` containing a string, and a field `candidates` containing a list of candidate JSON objects. Each candidate JSON object contains a field `identifier`, and a field `name`.

The reply JSON object may also contain a field `vote`, containing a JSON object with vote information, as described in [Submitting vote information](#submitting-vote-information).

```
GET /vote/<franchise>

request headers:
  none

reply status:
  200: Vote information
  404: Nonexistent franchise

reply body:
  200: application/json: {
    "election": { "name": string, "question": string, runs: { "from": date, "to": date }, "system": <system> },
    "ballots": [ { "name": string, "candidates": [ { "identifier": <candidate>, "name": string } ] } ],
  ? "vote": { "ballot": <ballot>, "candidates": [ <candidate> ] }
  }
  404: none
```

### Submitting vote information

The request body must contain a JSON object with the mime type `application/json`, containing the fields `ballot`, a ballot identifier, and `candidates`, a list of candidate identifiers. Candidate identifiers do not need to come from the same ballot as is submitted as `ballot`, but must all be valid candidates in the election the ballot and franchise is defined in relation to.

```
POST /vote/<franchise>

request headers:
  none

request body:
  application/json: { "ballot": <ballot>, "candidates": [ <candidate> ] }

reply status:
  201: Vote submitted
  204: Vote updated
  403: Vote could not be submitted or updated
  404: Nonexistent franchise

reply body:
  none except:
    403: application/json: { "error": <reason> }
```
