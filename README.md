# 🛠️ DamlForge 🛠️ 
Crowdfunding is an application built in daml to allow to collaboratively fund projects.

## I. Overview 

This project adopts and exemplifies the `proposal-accept` design pattern. 

Users can create a project contract and send contribution requests to parties. These parties can either accept or reject the request. When accepted, their contribution will be recorded in the project. There is a deadline after which contribution cannot be sent. Once the deadline is reached, the owner of the project can launch their project, and thus sending coupons of participations to contributors. These coupons will account for the fact that the project has successfully been funded or not, that is, the requested threshold has been met. The coupons can be redeemed by contributors. They are proof of their contribution.

This project contains 4 modules:
    1. Utils proposes utility functions around _Action_.
    2. Data provides various data types and type class instances and definitions.
    3. Crowfunding provides the 3 templates described below.
    4. Main contains test scenarii.

## II. Templates and choices

### 1. _CFProject_ contract

This represents a project that is currently being funded. It belongs to a certain owner, has a certain name, a certain deadline and threshold. It also tracks the current contributions and the list of parties that have been contacted to contribute.

There are several choices that can be exercised here:
- **nonconsuming** _GetRequestsStatus_ allows the owner to check the status of each requests that were sent to users.
- **nonconsuming** _GetProjectStatus_ allows the owner to see the funding status of their project.
- _Contribute_ allows users that were requested for contribution to contribute. They can do it several times.
- **postconsuming** _Launch_ allow the owner to launch their project. The outcome depends on the result of _GetProjectStatus_. After the dealine has been reached, this will create instances of _Coupon_ for each contributor.
- _SendRequests_ allows the owner to create instances of _ContributionRequest_ for several parties.

### 2. _ContributionRequest_ contract

This represents a request that has been sent by a project owner to a party.

There are two choices that can be exercised here:
- _Accept_ allows the target of the request to fund the project with an arbitrary amount.
- _Reject_ allows the target of the request to ignore the request and not fund the project.

### 3 _Coupon_ contract

This is the proof that a certain party has indeed contributed to a project.

There is a single choice to be exercised here:
- _Redeem_ to redeem the coupon. It does nothing (other than archiving the contract). This is left for addition use cases.

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

This typical workflow is named _typical_ and can be found in _Main.daml_.

## IV. Challenge(s)
sandbox-options:
  - --static-time
* `controller ... can` syntax causes warning in Daml 2.0+. The code itself does not cause any issues/errors in 2.5.0 but according to the warning, the syntax will be deprecated in the future versions of Daml. More information [here](https://docs.daml.com/daml/reference/choices.html#daml-ref-controller-can-deprecation).
* The new controller syntax requires a controller to be an observer first before they can exercise a choice, otherwise it'll throw an error: "Attempt to fetch or exercise a contract not visible to the committer." For more information, check out [this post](https://discuss.daml.com/t/error-attempt-to-fetch-or-exercise-a-contract-not-visible-to-the-committer/1304/1) on the Daml Forum.
* The project was created by using `empty-skeleton` and the following was removed from `daml.yaml`:
```
sandbox-options:
   - --wall-clock-time
```
and the following was added:

```
exposed-modules:
  - Main
```
For more info, check out [this post](https://discuss.daml.com/t/sandbox-options-wall-clock-time/5692/16?u=cathy_jung) on Daml Forum and [Daml Doc](https://docs.daml.com/tools/navigator/index.html?&_ga=2.48248804.337210607.1673989679-241632404.1672853064&_gac=1.17025355.1673455980.CjwKCAiA2fmdBhBpEiwA4CcHzfI2w1_D95zAr3_d6QTypOMXGTpUxtS06c55inucNwZvUZn4AebsJxoCZEgQAvD_BwE&_gl=1*elem6v*_ga*MjQxNjMyNDA0LjE2NzI4NTMwNjQ.*_ga_GVK9ZHZSMR*MTY3Mzk5NDQzOS4zMS4xLjE2NzM5OTQ3MDcuMC4wLjA.#logging-in-as-a-party).


## V. Compiling & Testing
To compile and test, run the pre-written script in the `Test.daml` under /daml OR run:
```
$ daml start
```