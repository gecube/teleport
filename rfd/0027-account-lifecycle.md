---
authors: Sasha Klizhentas (sasha@goteleport.com)
state: draft
---

# RFD 27 - Account Lifecycle: Recovery and Cancellation

## What

Introduces cloud account recovery and cancellation flow.

## Why

Having a secure self-management account recovery and cancelation
flow improves user experience and reduces support load on teams.

## Details

The recovery flow focuses on local accounts.
SSO account recovery is out of scope, however UI will provide some hints.

Each local user will receive 3 account recovery tokens on signup.

Each token is a crypto-randomly generated 32 byte string that could be used only once.
Once signed up, users will be presented with the recovery tokens printed on the
screen with a message:

```
Please save these account recovery tokens in a safe offline place.
You can use each token once if you loose your second factor.

* recover-example-teleport-sh-token-1
* recover-example-teleport-sh-token-2
* recover-example-teleport-sh-token-3

```

The cloud prefix is for customer convenience in case if they forgot the domain
name as well.

### Recovery scenarios

The first two recovery scenarios work for every system user, reducing the load
on Teleport system administrators as well.

**User lost a second factor**

If user lost a second factor, but not password, they can recover using
recovery token:

* Login with a username and password
* Instead of pressing second factor, enter a one-time recovery token
* Enroll new second factor right away.

The UI flow should prompt a blocking request for a second factor right
after user has entered a valid recovery token.

* The used account recovery token is no longer valid. The new recovery tokens are printed to the user.

This flow ensures two factors are verified: the valid password and a recovery token.

**User lost a password**

If user lost a password, but not a second factor, they could recover using
a recovery link sent to their email and a second factor:

* User enters their email and clicks forgot password.
* The recovery system sends a short-lived reset short lived password link to the email and
prints (in all cases, whether the email is valid or invalid):

`Please check your inbox for a recovery link and follow the instructions.`

The link includes a crypto-random short-lived 32 byte token.

* Once user clicks on the link, they have to present a valid second factor and a
recovery token, after that they will be able to reset their password.

This flow ensure that two factors are verified:

* Second factor TOTP token or U2F device
* A valid account recovery token.

Notice that we do not consider a valid email a valid verification factor,
although it's a nice-to-have protection layer.

* The used account recovery token is no longer valid. The new recovery tokens are printed to the user.

**Internal request for cancelling the account**

To request account cancellation, a user with sufficient privileges should:

* Be logged into Teleport UI.
* Press a "cancel my account" button in the UI.
* UI should verify that they own a second factor.
* Verify that they have write permission on the billing.

```
We have marked your account as inactive. In 10 days, we are going to delete the account and all it's data.
To prevent this, log into Teleport and press "revert account cancelation".
```

If anyone logs into this account, we should show them the warning:

```
Your account was marked as inactive at the request of `email`.
To prevent account from being deleted, log into Teleport and press "revert account cancelation".
```

Only users with write permissions to billing should be able to revert account
cancelation. We should asking a user for second factor proof to revert account cancelation to prevent
a possible account overtake by a malicious actor who wishes to keep the account for accessing
the resources.

The possible vectors of attack is for malicious actor to overtake someone's
account with a request to delete all the data. The grace period and active privileged
user should mitigate against that.

**External request for canceled account**

If the user used their @gmail address, they won't be able to recover their account with us,
but can request cancellation.

* Request account cancellation by clicking on "cancel my account link" URL.
* The user should have write privileges on the billing information for the cancellation to work.
* We email crypto random 32 byte hex cancel account link to their email address.
* They click the link and we mark the account as "inactive".

```
We have marked your account as inactive. In 10 days, we are going to delete the account and all it's data.
To prevent this, log into Teleport and press "revert account cancelation".
```

If anyone logs into this account, we should show them the warning:

```
Your account was marked as inactive at request of `email`.
If you would like to revert acount from being canceled in x days, go to billing page
and press "revert account cancelation".
```

All users with write billing permission in the email account should get the following notification over email:

```
Your account was marked as inactive at request of `email`.
The account and all data will be deleted after 10 days grace period.
To prevent this, login using your credentials and press "revert account cancelation".
```

Only users with write permissions to billing should be able to revert account
cancelation. We should asking a user for second factor proof to revert account cancelation to prevent
a possible account overtake by a malicious actor who wishes to keep the account for accessing
the resources.

The possible vectors of attack is for malicious actor to overtake someone's
email account and launch an attack to delete the data. The grace period and the broadcast
notification should allow users to mitigate the deletion by logging in their account.

We should recommend users to have at least two users with write billing permissions
so the other person will be notified and can act.


**When all is lost**

In case if a system administrator has lost everything, we can not automate the recovery flow.
For social emails @gmail.com and so on, we can't have any connection between a user
and their account, so they only will be able to request cancel of their account using
external flow.


If it's an organization whose users have @corp domain, we can ask for advanced slow validation
using offline methods:

* The company acme.com should proove the ownership of their domain
by posting their registered U.S. corporate mail address in the TEXT record:

teleport.acme.com IN TXT <corp address, signing officer, verifying officer>

Our support should verify that this address matches their registered office location
in the company official registration documents and is not a P.O. Box.

The verifying officer will receives a first mail with a notification that signing officer
has requested account reset. Once support gets delivery confirmation,
They send an overnight account reset code with a signing officer signature required.

Overnight account reset codes will allow a user to create a privileged user
using our standard user invite flow.

A possible vector of attack is a malicious actor, who overtakes coroprate DNS,
and impersonates the signing officer at the office location. To prevent that,
verifying officer would receive another mail with a notification before the signing
officer gets a reset email. So two parties would have to be impersonated.

### Flows

[Github account recovery](https://docs.github.com/en/github/authenticating-to-github/recovering-your-account-if-you-lose-your-2fa-credentials)
