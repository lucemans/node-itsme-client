[![npm version](https://img.shields.io/npm/v/itsme-client.svg?style=for-the-badge)](https://www.npmjs.com/package/itsme-client)

# Itsme® client

This library's purpose it to make your server's communication with itsme®
more pleasant.

_itsme-client_ discovers the OpenID configuration of itsme and allows you to
easily exchange tokens without worrying about fetching, caching, signing and
encrypting.

# Features

    * Endpoint discovery
    * Generating an Itsme URL
    * Exchanging an Authorization Token
    * Extracting claims from an ID Token
    * Getting claims from the User Info endpoint
    * Extracting your public keys as a JWK Set
    * Decrypting and verifying JWTs
    * Encrypting and signing JWTs
    * Key rollover
    * Normalizing returned values

The library is written in TypeScript, so typings are available. Plain Node.js
will also work.
When using TypeScript, add `@node_modules/itsme-client/@types` to your
[typeRoots](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html#types-typeroots-and-types).

# Usage

## General Outline

The generic login flow is implemented as follows

- The user is directed to a link generated by `generateAuthUrl`
- The user performs their login on the itsme-site
- The user is redirected back to us landing on the appropriate `redirectUrl` depending on the `serviceCode` used.
- We extract the `authCode` from the url, and exchange this for a token with the `itsme` servers
- We use this token to request further information about the user 

## Initialize ItsmeClient

This is the basic usage, more options and methods are available. Intellisense and jsdoc should
help you find and understand them.

```typescript
import { createKeyStore, IdentityProvider, ItsmeClient } from 'itsme-client';

async function initItsmeClient() {
    const itsmeDiscoveryUrl = 'https://e2emerchant.itsme.be/oidc/.well-known/openid-configuration';
    const itsmeProvider = await IdentityProvider.discover(itsmeDiscoveryUrl);
    return new ItsmeClient(itsmeProvider, {
        clientId: 'your client id here',
        keyStore: await createKeyStore(yourJwkSet),
        serviceCodes: {
            YOUR_SERVICE_CODE: 'https://the-redirect-url-matching-this-service-code',
        },
    });
}
```

## Redirecting user to Itsme login

When we want to initialize a session with our user we are required to redirect them to the itsme portal where they can perform the login.
The following function will generate the url to which the user should be redirected.
Upon completion the user will be directed back to the `redirectURL` that matched your `serviceCode`.
Then proceed with [processing the auth-code](#obtaining-user-info-with-an-authorization-token)

```typescript
import { ItsmeClient } from 'itsme-client';

async function getRedirectToItsmeLink(itsmeClient: ItsmeClient) {
    const redirectURL = client.generateAuthUrl({
        // The service code
        service: "YOUR_SERVICE_CODE",
        // Additional Query Parameters
        additionalParams: {},
        // Addition scopes, for ex. the profile scope
        additionalScopes: ['profile'],
        // The state you want to pass in with your request
        state: 'YOUR_RANDOM_STATE_HERE_HERE'
    })

    return redirectURL;
}
```

## Obtaining user info with an Authorization token

```typescript
import { ItsmeClient } from 'itsme-client';

async function wrapper(itsmeClient: ItsmeClient) {
    const token = await itsmeClient.exchangeAuthorizationCode(
        'Authorization code here',
        itsmeClient.getRedirectUri('YOUR_SERVICE_CODE'),
    );

    // Get the user info via the userInfo endpoint
    const userInfo = await itsmeClient.userInfoComplete(token.access_token);

    // Same thing with intermediary steps
    const userInfoJwt = await itsmeClient.userInfo(accessToken);
    const decryptedUserInfo = await itsmeClient.decryptUserInfo(userInfoJwt);
    const userInfoStepByStep = await itsmeClient.verifyUserInfo(decryptedUserInfo);


    // Or get the claims via the ID Token
    const idTokenPayload = await itsmeClient.decryptAndVerifyIdToken(token.id_token);
}
```

## Extracting your public keys as a JWK Set

This library supports extracting your public keys as a JWK Set for easy exposure
via URL or other means.

```typeScript
itsmeClient.getPublicJwkSet();
```

# Responses

RESPONSES.md contains examples of responses of certain methods.
