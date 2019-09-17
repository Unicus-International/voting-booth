# Terminology used in Voting Booth

## Roles

### User

### Commissioner

### Comptroller

### Elector

## Concepts

### Electoral roll

An electoral roll is a collection of potential electors, attached to a user account. These potential electors can then be made into electors for a given election, without having to maintain the roll or rolls externally.

```
electoralRoll: [ (
  name: string,
  email: string,
) ]
```

### Election

An election is a question on which an electorate (i.e. a collection of electors) vote, a collection of one or more ballots and their associated candidates, and the associated process control measures such as start and end dates, the franchies to be exercised, the choice vote counting system, &c.. Each election happens at most once, and once completed cannot be reopened. The result of an election might be a single selected candidate, or a collection of selected candidates, orderered or unordered.

```
election: (
  identifier: <election>,
  ballots: [ <ballot> ],
  candidates: [ <candidate> ],
  franchises: [ <franchise> ],

  runs: ( from: date, to: date ),
  system: <system>,
  votes: [ <vote> ],

  commissioner: <commissioner>,
  comptrollers: [ <comptroller> ],
  electors: [ <elector> ],
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
