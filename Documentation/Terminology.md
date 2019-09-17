# Terminology used in Voting Booth

## Concepts

### User

A user is a person within the user system, with an email address as their user ID. Users may also have a display name set, and if they registered themselves (see [Comptroller](#comptroller) below) they will also have a password set.

```
user: (
  email: <email>,
  name: string?,
  password: password?,
)
```

### Electoral roll

An electoral roll is a collection of potential electors, attached to a user account. These potential electors can then be made into electors for a given election, without having to maintain the roll or rolls externally.

```
electoralRoll: [ (
  name: string,
  email: string,
) ]
```

### Election

An election is a question on which an electorate (i.e. a collection of electors) vote, a collection of one or more ballots and their associated candidates, and the associated process control measures such as start and end dates, the franchises to be exercised, the choice vote counting system, &c.. Each election happens at most once, and once completed cannot be reopened. The result of an election might be a single selected candidate, or a collection of selected candidates, ordered or unordered.

```
election: (
  identifier: <election>,

  name: string,
  question: string,

  ballots: [ <ballot> ],
  candidates: [ <candidate> ],
  franchises: [ <franchise> ],

  runs: ( from: date, to: date ),
  system: <system>,
  votes: [ <vote> ],

  commissioner: <user>,
  comptrollers: [ <user> ],
)
```

### Candidate

A candidate is a potential answer in the question of an election. Candidates need not be people, but can, depending on the question of the election, be anything to be selected between. In the simplest case, the candidates may simply be "Yes" and "No".

```
candidate: (
  identifier: <candidate>,
  election: <election>,
  name: string,
)
```

### Franchise

A franchise is a claim for a single ballot in a single, specific election. In single-ballot elections, this is roughly equivalent to the ballot itself.

```
franchise: (
  identifier: <franchise>,
  election: <election>,
)
```

### Ballot

A ballot is a collection of candidates, which may be ordered or unordered. A ballot is associated with a single, specific election.

```
ballot: (
  identifier: <ballot>,
  election: <election>,
  candidates: [ <candidate> ],
)
```

### Vote

A vote is an answer to the question of the election. A vote is connected to the franchise that was discharged to cast it, and the ballot on which it was cast, as well as an ordered list of candidates selected from the ballot. Depending on the vote counting systems, this list may only have one entry (the selected candidate), or it may list some or all of the candidates on the ballot in order of preference, or indeed candidates from other ballots.

```
vote: (
  election: <election>,
  franchise: <franchise>,
  ballot: <ballot>,
  candidates: [ <candidate> ],
)
```

## Roles

Roles are representative of tasks users or guests can fulfill. A user may fulfill any number of roles at any given time, with regards to different elections.

### Commissioner

A commissioner is a user who has opened and configured (commissioned) an election. Commissioners are also comptrollers with regards to any elections they have commissioned.

### Comptroller

A comptroller is a user who has been selected to help oversee an election. They do not need to be registered to do so, but if they are not registered, shadow accounts are created for them for organizational purposes. If they later register as full users, these accounts are fully joined.

### Elector

An elector is a guest who is enfranchised to vote in a particular election. Electors are identified only by their franchise identifier, in order to preserve anonymity. Commissioners and comptrollers are often, but not by necessity, electors.
