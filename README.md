# 🛠️ MM Crowd Funding 🛠️ 
MM Crowd Funding is an application built in daml to allow to collaboratively fund projects.

## I. Overview 

This project adopts and exemplifies the `proposal-accept` design pattern. 

Users can create a project contract and send contribution requests to parties. These parties can either accept or reject the request. When accepted, their contribution will be recorded in the project. There is a deadline after which contribution cannot be sent. Once the deadline is reached, the owner of the project can launch their project, and thus sending coupons of participations to contributors. These coupons will account for the fact that the project has successfully been funded or not, that is, the requested threshold has been met. The coupons can be redeemed by contributors. They are proof of their contribution.

This project contains 4 modules:
    1. Utils proposes utility functions around `Action`.
    2. Data provides various data types and type class instances and definitions.
    3. Crowfunding provides the 3 templates described below.
    4. Endpoints creates wrappers around submitting transactions
    5. Main contains test scenarii.

## II. Templates and choices

### 1. `CFProject` contract

This represents a project that is currently being funded. It belongs to a certain owner, has a certain name, a certain deadline and threshold. It also tracks the current contributions and the list of parties that have been contacted to contribute.

There are several choices that can be exercised here:
- **nonconsuming** `GetRequestsStatus` allows the owner to check the status of each requests that were sent to users.
- **nonconsuming** `GetProjectStatus` allows the owner to see the funding status of their project.
- `Contribute` allows users that were requested for contribution to contribute. They can do it several times.
- **postconsuming** `Launch` allow the owner to launch their project. The outcome depends on the result of `GetProjectStatus`. After the dealine has been reached, this will create instances of `Coupon` for each contributor.
- `SendRequests` allows the owner to create instances of `ContributionRequest` for several parties.

### 2. `ContributionRequest` contract

This represents a request that has been sent by a project owner to a party.

There is one choice that can be exercised here:
- `Respond` allows the target of the request to accept or refuse it depending on the passed parameter.

### 3 `Coupon` contract

This is the proof that a certain party has indeed contributed to a project.

There is a single choice to be exercised here:
- `Redeem` to redeem the coupon. It does nothing (other than archiving the contract). This is left for addition use cases.

## III. Typical workflow

  1. _Alice_ creates a CFProject named _AliceProject_ with a deadline in 10 days and a threshold of 1000.
  2. _Alice_ contributes herself to the project for 400.
  3. _Alice_ sends contribution requests to _Bob_, _Judith_ and _Jacob_.
  4. _Judith_ is not interested and refuses the proposal.
  5. _Bob_ accepts and gives 500.
  6. _Alice_ checks the current status of her requests.
  7. _Jacob_ accepts and gives 300.
  8. _Alice_ checks the current status of her project.
  9. Once the deadline has passed, _Alice_ launches the project.
  10. The 3 contributors redeem their coupons.

This typical workflow is named _typical_ and can be found in _Main.daml_.

## IV. Challenge(s)

### Unable to play with time in _Navigator_

I was unable tu run `daml start` When using `setTime`. 
I looked up many resources and discussions online, but none of them seemed to work.
A solution that came up many time was to use
    sandbox-options:
    - --static-time
But the option is not recognised by the sandbox.
As a consequence, it is not possible tu exercise the `Launch` choice on a crowdfunded projects in my example using `Navigator` since the deadline will never occur.

### Permission for exercising `GetProjectStatus`

I would have wanted everybody among contributors to be able to exercise this choice. However, I did not know how to express as a controller "anyone among those". I believe it is not possible. The closest I could find was to pass the party that exercises the choice as parameter as the choice itself, and ensuring within the body of the choice that this party indeed belongs to contributors or requested contributors. But I eventually settled on only allowing the owner to exercise this choice. I guess that only observers could exercise this choice anyway, thus rendering the previous option more potent, but I did not feel like passing an extra parameter just to set the controller (which I did for contributions). I would be interested to discuss more about these permissions and associated patterns.

## V. Compiling & Testing

To launch the test suite, run
```
$ daml test
```
To initialize some users and play with the contracts in _Navigator_, use
```
$ daml start
```