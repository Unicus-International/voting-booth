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

Logout accepts no parameters. The reply will have a status code of 204, and the body of the response will be empty.

```
GET /user/logout

request headers:
  X-Session-Id: <session>

reply status:
  204: Logout successful
```
