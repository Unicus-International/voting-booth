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

The reply body must contain a JSON object that contains a field `election`, containing the fields `name`, a string; `question`, a string; `runs`, a JSON list containing a start and and end date, both ISO8601-encoded strings; and `system`, string containing an identifier for the system in use, as well as a field `ballots` which contains a list of ballots, each a JSON object with a field `name` containing a string, a field `identifier` containing an identifier, and a field `candidates` containing a list of candidate JSON objects. Each candidate JSON object contains a field `identifier`, and a field `name`.

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
    "election": {
      "name": string,
      "question": string,
      "runs": [ date, date ],
      "system": <system>,
      "updatableVotes": boolean,
      "ballots": [
        {
          "name": string,
          "identifier": <ballot>,
          "candidates": [
            {
              "identifier": <candidate>,
              "name": string
            }
          ]
        }
      ]
    },
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

## Election comissioning

All calls to the APIs outlined below require the header `X-Session-Id` to be set, containing a valid session ID.

### Listing comissioned elections

When called with a valid session ID, the response will contain a JSON list of objects, each containing an election identifier and the name of the corresponding election.

```
GET /elections/list

request headers:
  X-Session-Id: Session ID

reply status:
  200: Request processed
  403: Not logged in

reply body:
  200: application/json: [
    {
      "identifier": <election>,
      "name": string
    }
  ]
  403: none
```

### Getting the details of an election

When called with a valid session ID, the response will contain a JSON object, containing a description of the selected election, as is available to a commissioner or comptroller.

```
GET /elections/<election>/list

request headers:
  X-Session-Id: Session ID

reply status:
  200: Request processed
  403: Not logged in
  404: Nonexistent election

reply body:
  none except:
    200: application/json: {
      "name": string,
      "question": string,
      "runs": [ date, date ],
      "system": <system>,
      "updatableVotes": boolean,
      "ballots": [
        {
          "name": string,
          "identifier": <ballot>,
          "candidates": [
            {
              "identifier": <candidate>,
              "name": string
            }
          ]
        }
      ]
    }
```

### Listing electoral systems

When called, returns a list of available electoral systems (methods for gathering and counting votes).

```
GET /elections/systems/list

reply status:
  200: Request processed

reply body:
  200: application/json: [
    {
      "identifier": <system>,
      "name": string,
      "description": string
    }
  ]
```

### Commissioning a new election

```
POST /elections/commission

request headers:
  X-Session-Id: Session ID

request body:
  application/json: {
    "name": string,
    "question": string,
    "updatableVotes": boolean,
    "runs": [ date, date ],
    "system": <system>
  }

reply status:
  201: Election created
  403: Not logged in

reply body:
  none except:
    201: as GET /elections/<election>/list
```

### Listing election ballots

When called with a valid session ID, will list the ballots in an election.

```
GET /elections/<election>/ballots/list

request headers:
  X-Session-Id: Session ID

reply status:
  200: Request processed
  403: Not logged in
  404: Nonexistent election

reply body:
  none except:
    200: application/json: [
      {
        "name": string,
        "identifier": <ballot>
      }
    ]
```

### Listing a specific ballot

```
GET /elections/<election>/ballots/<ballot>/list

request headers:
  X-Session-Id: Session ID

reply status:
  200: Request processed
  403: Not logged in
  404: Nonexistent election or nonexistent ballot

reply body:
  none except:
    200: application/json: {
      "name": string,
      "identifier": <ballot>,
      "candidates": [
        {
          "identifier": <candidate>,
          "name": string
        }
      ]
    }
```

### Creating a new ballot

```
POST /elections/<election>/ballots/new

request headers:
  X-Session-Id: Session ID

request body:
  application/json: {
    "name": string,
    "candidates": [
      {
        "name": string
      }
    ]
  }

reply status:
  201: Ballot created
  403: Not logged in
  404: Nonexistent election
  409: The election has been opened

reply body:
  none except:
    201: as GET /elections/<election>/ballots/<ballot>
```
