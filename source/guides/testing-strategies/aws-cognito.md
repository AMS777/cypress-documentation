---
title: Amazon Cognito Programmatic Authentication
---

{% note info %}
# {% fa fa-graduation-cap %} What you'll learn
- Programmatically authenticate with {% url "Amazon Cognito" https://aws.amazon.com/cognito %} via a custom Cypress command
- Adapting your {% url "Amazon Cognito" https://aws.amazon.com/cognito %} application for programmatic authentication during testing

{% endnote %}

## What is Amazon Cognito?

{% url "Amazon Cognito" https://aws.amazon.com/cognito %} is an authentication provider apart of {% url "Amazon Web Services (AWS)" https://aws.amazon.com %}.  It "lets you add user sign-up, sign-in, and access control to your web and mobile apps quickly and easily" and scales to millions of users and supports sign-in with social identity providers, such as Facebook, Google, and Amazon, and enterprise identity providers via SAML 2.0."

Testing applications deployed with {% url "Amazon Cognito" https://aws.amazon.com/cognito %} can use different strategies; mocking and testing the full stack.

## Programmatic Authentication with Amazon Cognito

The documentation for {% url "Amazon Cognito" https://aws.amazon.com/cognito %} recommends using the 
{% url "AWS Amplify Framework Authentication Library" https://aws-amplify.github.io/amplify-js/api/classes/authclass.html %} from the {% url "AWS Amplify Framework" https://aws.amazon.com/amplify/framework/ %} to interact with a deployed {% url "Amazon Cognito" https://aws.amazon.com/cognito %} instance.

Using the {% url "AWS Amplify Framework Authentication Library" https://aws-amplify.github.io/amplify-js/api/classes/authclass.html %}, we are able to programmatically drive the creation and authentication of users against a fully deployed back end.

This illustrates the limited code from the {% url "AWS Amplify Framework" https://aws.amazon.com/amplify/framework/ %} needed to programmatically log an existing a user into an application.

```jsx
// Add 'aws-amplify' library into your application

// Configure Auth category with your Amazon Cognito credentials
Amplify.configure({
  Auth: {
    identityPoolId: 'XX-XXXX-X:XXXXXXXX-XXXX', // Amazon Cognito Identity Pool ID
    region: 'XX-XXXX-X', // Amazon Cognito Region
  }
})

// Call Auth.signIn with user credentials
Auth.signIn(username, password)
  .then((user) => console.log(user))
  .catch((err) => console.log(err))
```

## Amazon Cognito Setup

If not already setup, you will need to [create an account][awsCreateAccount] with [Amazon Web Services][aws].

An implementation for use with [AWS Cognito](https://github.com/cypress-io/cypress-realworld-app/blob/develop/src/containers/AppCognito.tsx) is available in the [Cypress Real World App][cypressrwa].

Clone the [Cypress Real World App][cypressrwa] and install the [AWS Amplify CLI][awsamplifycli] as follows:

```jsx
npm install -g @aws-amplify/cli
```

The [Cypress Real World App][cypressrwa] is configured with an optional [AWS Cognito][cognito] instance via the [Amplify Authentication][amplifyauth] library.

The [Amplify CLI][awsamplifycliconfig] is used to provision the [AWS][aws] infrastructure needed to configure your environment and cloud resources.

First, run the [amplify init][amplifyinit] command to initialize the [Cypress RWA][cypressrwa].  This will provision the project with your [AWS][aws] credentials.

```jsx
amplify init
```

Next, run the [amplify push][amplifypush] command to create the [AWS Cognito][cognito] resources in the cloud:

```jsx
amplify push
```

## Setting Amazon Cognito app credentials in Cypress

First, we need to configure Cypress to use the {% url "AWS Cognito" https://aws.amazon.com/cognito %} environment variables set in `.env` inside of the `cypress/plugins/index.js` file. In addition, we are using the `aws-exports.js` supplied during the [AWS Amplify][awsamplify] build process.

```jsx
// cypress/plugins/index.js
// initial imports ...

const awsConfig = require(path.join(__dirname, '../../aws-exports-es5.js'))

dotenv.config()

export default (on, config) => {
  // ...
  config.env.cognito_username = process.env.AWS_COGNITO_USERNAME
  config.env.cognito_password = process.env.AWS_COGNITO_PASSWORD
  config.env.awsConfig = awsConfig.default

  // plugins code ...

  return config
}
```
## Custom Command for Amazon Cognito Authentication

Next, we'll write a command to perform a programmatic login into [Amazon Cognito][cognito] and set items in localStorage with the authenticated users details, which we will use in our application code to verify we are authenticated under test.

In this `loginByCognitoApi` command, we call `Auth.signIn`, then use that response to set the items inside of localStorage for the UI to know that our user is logged into the application.

```jsx
// cypress/support/auth-provider-commands/cognito.ts

import Amplify, { Auth } from 'aws-amplify'

Amplify.configure(Cypress.env('awsConfig'))

// Amazon Cognito
Cypress.Commands.add('loginByCognitoApi', (username, password) => {
  const log = Cypress.log({
    displayName: 'COGNITO LOGIN',
    message: [`🔐 Authenticating | ${username}`],
    // @ts-ignore
    autoEnd: false,
  })

  log.snapshot('before')

  const signIn = Auth.signIn({ username, password })

  cy.wrap(signIn, { log: false }).then((cognitoResponse) => {
    const keyPrefixWithUsername = `${cognitoResponse.keyPrefix}.${cognitoResponse.username}`

    window.localStorage.setItem(
      `${keyPrefixWithUsername}.idToken`,
      cognitoResponse.signInUserSession.idToken.jwtToken
    )

    window.localStorage.setItem(
      `${keyPrefixWithUsername}.accessToken`,
      cognitoResponse.signInUserSession.accessToken.jwtToken
    )

    window.localStorage.setItem(
      `${keyPrefixWithUsername}.refreshToken`,
      cognitoResponse.signInUserSession.refreshToken.token
    )

    window.localStorage.setItem(
      `${keyPrefixWithUsername}.clockDrift`,
      cognitoResponse.signInUserSession.clockDrift
    )

    window.localStorage.setItem(
      `${cognitoResponse.keyPrefix}.LastAuthUser`,
      cognitoResponse.username
    )

    window.localStorage.setItem('amplify-authenticator-authState', 'signedIn')
    log.snapshot('after')
    log.end()
  })

  cy.visit('/')
})
```


Finally, we can use our `loginByCognitoApi` command in at test.  Below is our test to login as a user via [Amazon Cognito][cognito], complete the onboarding process and logout.

{% note success Runnable Test %}
The [runnable version of this test](https://github.com/cypress-io/cypress-realworld-app/blob/develop/cypress/tests/ui-auth-providers/cognito.spec.ts) is in the [Cypress Real World App][cypressrwa].
{% endnote %}

```jsx
describe('Cognito', function () {
  beforeEach(function () {
    cy.task('db:seed')
    cy.loginByOktaApi(Cypress.env('cognito_username'), Cypress.env('cognito_password'))
  })

  it('shows onboarding', function () {
    cy.contains('Get Started').should('be.visible')
  })
})
```

## Adapting an Amazon Cognito App for Testing

The [Cypress Real World App][cypressrwa] is used and provides configuration and runnable code for both the React SPA and the Express back end.

The front end uses the [Amplify Authentication Library][awsamplifyauth]. The back end uses the [express-jwt][expressjwt] to validate JWTs from [Amazon Cognito][cognito].

### Adapting the back end

In order to validate API requests from the frontend, we install [express-jwt](https://github.com/auth0/express-jwt) and [jwks-rsa](https://github.com/auth0/node-jwks-rsa) and configure validation for JWT's from [Amazon Cognito][cognito].

```jsx
// backend/helpers.ts
// ... initial imports
import jwt from 'express-jwt'
import jwksRsa from 'jwks-rsa'

// ...

const awsCognitoJwtConfig = {
  secret: jwksRsa.expressJwtSecret({
    jwksUri: `https://cognito-idp.${awsConfig.aws_cognito_region}.amazonaws.com/${awsConfig.aws_user_pools_id}/.well-known/jwks.json`,
  }),

  issuer: `https://cognito-idp.${awsConfig.aws_cognito_region}.amazonaws.com/${awsConfig.aws_user_pools_id}`,
  algorithms: ['RS256'],
}

export const checkCognitoJwt = jwt(awsCognitoJwtConfig).unless({ path: ['/testData/*'] })
```

Once this helper is defined, we can use globally to apply to all routes:

```jsx
// backend/app.ts
// initial imports ...
import { checkCognitoJwt } from './helpers'

// ...

if (process.env.REACT_APP_AWS_COGNITO) {
  app.use(checkCognitoJwt)
}

// routes ...
```
### Adapting the front end

We need to update our front end React app to allow for authentication with [Amazon Cognito][cognito] using the [Amplify Authentication Library][awsamplifyauth].

First, we create a `AppCognito.tsx` container, based off of the `App.tsx` component.

A `useEffect` hook is added to get the access token for the authenticated user and send an `COGNITO` event with the `user` and `token` objects to work with the existing authentication layer (`authMachine.ts`).  We use the `AmplifyAuthenticator` component to provide the login form from [Amazon Cognito][cognito].

```jsx
// src/containers/AppOkta.tsx
// initial imports ...
import Amplify from "aws-amplify";
import { AmplifyAuthenticator, AmplifySignUp, AmplifySignIn } from "@aws-amplify/ui-react";
import { AuthState, onAuthUIStateChange } from "@aws-amplify/ui-components";

import awsConfig from "../aws-exports";

Amplify.configure(awsConfig);

// ...

const AppCognito: React.FC = () => {

  // ...

  useEffect(() => {
    return onAuthUIStateChange((nextAuthState, authData) => {
      if (nextAuthState === AuthState.SignedIn) {
        authService.send("COGNITO", { user: authData });
      }
    });
  }, []);

  // ...

  return isLoggedIn ? (
    // ...
  ) : (
    <Container component="main" maxWidth="xs">
      <CssBaseline />
      <AmplifyAuthenticator usernameAlias="email">
        <AmplifySignUp slot="sign-up" usernameAlias="email" />
        <AmplifySignIn slot="sign-in" usernameAlias="email" />
      </AmplifyAuthenticator>
    </Container>
  );
};

export default AppCognito;
```

{% note success %}
Note: The full [AppCognito.tsx component](https://github.com/cypress-io/cypress-realworld-app/blob/develop/src/containers/AppCognito.tsx) is in the [Cypress Real World App][cypressrwa].
{% endnote %}

Next, we update our entry point (`index.tsx`) to use our `AppCognito.tsx` component.

```jsx
// src/index.tsx
// ... initial imports
import AppCognito from "./containers/AppCognito"

// ...

if (process.env.REACT_APP_AWS_COGNITO) {
  ReactDOM.render(
    <Router history={history}>
      <ThemeProvider theme={theme}>
        <AppCognito />
      </ThemeProvider>
    </Router>,
    document.getElementById("root")
  );
}
```


[cypressrwa]: https://github.com/cypress-io/cypress-realworld-app
[cypressrecipes]: https://github.com/cypress-io/cypress-example-recipes
[cypresscommands]: https://on.cypress.io/api/commands
[cypresstask]: https://on.cypress.io/api/task
[cypressfixture]: https://on.cypress.io/api/fixture
[aws]: https://aws.amazon.com
[awsCreateAccount]: https://docs.amplify.aws/start/getting-started/installation/q/integration/react#sign-up-for-an-aws-account
[cognito]: https://aws.amazon.com/cognito
[awsamplifyframework]: https://aws.amazon.com/amplify/framework/ 
[awsamplifyauth]: https://aws-amplify.github.io/amplify-js/api/classes/authclass.html
[awsamplifycli]: https://docs.amplify.aws/cli
[awsamplifycliconfig]: https://docs.amplify.aws/cli/start/install#configure-the-amplify-cli
[amplifyauth]: https://docs.amplify.aws/cli/auth/overview 
[amplifypush]: https://docs.amplify.aws/cli/start/workflows#amplify-push 

{% url "" https:// %}
{% url "Amazon Cognito" https://aws.amazon.com/cognito %}
{% url "AWS Amplify Framework Authentication Library" https://aws-amplify.github.io/amplify-js/api/classes/authclass.html %}
{% url "AWS Amplify Framework" https://aws.amazon.com/amplify/framework/ %}
