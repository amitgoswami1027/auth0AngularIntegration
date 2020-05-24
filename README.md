
## AUTH0 INTEGRATION 
### Angular: Authentication : Our app's basic authentication should include:
* Login and logout
* User profile and token management
* Session persistence
* Authorization of HTTP requests with access token

#### STEP-01 : Install Auth0.js
*  npm install auth0-js@latest --save
#### STEP-02 : Dynamic Environment Configuration
Let's create a file to store information about our app's environment. We're currently developing on localhost:4200, but the app will be deployed on the Node server eventually, and in production, it will run on a reverse proxy. We'll need to make sure our development environment doesn't break our production environment and vice versa. Create a folder: src/app/core, then add a file there called env.config.ts:
```
/ src/app/core/env.config.ts
const _isDev = window.location.port.indexOf('4200') > -1;
const getHost = () => {
  const protocol = window.location.protocol;
  const host = window.location.host;
  return `${protocol}//${host}`;
};
const apiURI = _isDev ? 'http://localhost:8083/api/' : `/api/`;

export const ENV = {
  BASE_URI: getHost(),
  BASE_API: apiURI
};

```
Another way to do this would be to set up your environments/environment.*.ts files with environment-dependent settings.
#### STEP-03 : Authentication Configuration
We'll store our Auth0 authentication configuration in an auth.config.ts file. Create the following blank file: src/app/auth/auth.config.ts.
```
import { ENV } from './../core/env.config';

interface AuthConfig {
  CLIENT_ID: string;
  CLIENT_DOMAIN: string;
  AUDIENCE: string;
  REDIRECT: string;
  SCOPE: string;
};

export const AUTH_CONFIG: AuthConfig = {
  CLIENT_ID: '[AUTH0_CLIENT_ID]',
  CLIENT_DOMAIN: '[AUTH0_CLIENT_DOMAIN]', // e.g., you.auth0.com
  AUDIENCE: '[YOUR_AUTH0_API_AUDIENCE]', // e.g., http://localhost:8083/api/
  REDIRECT: `${ENV.BASE_URI}/callback`,
  SCOPE: 'openid profile'
};
```
#### STEP-04 : Authentication Service
* Authentication logic on the front end will be handled with an AuthService authentication service. Now open the generated 
  auth.service.ts file and add the necessary code to our authentication service. This service uses the config variables from 
  auth.config.ts to instantiate an auth0.js WebAuth instance. An RxJS BehaviorSubject is used to provide a stream of authentication 
  status events that we can subscribe to anywhere in the app.
* The constructor checks the app authentication status upon initialization. If the user has not logged out of our Angular app from a 
  previous session (their token has not expired), we'll call a method called renewToken() to verify that their Auth0 session on the 
  authentication server is also still valid. If it is, we'll receive a fresh access token.
* Note: The Angular app's authentication lifespan is not the same thing as the authentication session on the server. We manage the 
  Angular app authentication with the expiration of the JWT access token, and the persistence or removal of this tells us whether or not 
  to ask the server if the user's Auth0 authentication session is still valid when initializing the app.
* We'll receive an an access_token and a time until token expiration (expiresIn) from Auth0 when returning to our app. The handleAuth() 
  method uses Auth0's parseHash() method callback to get the user's profile (_getProfile()) and set the session (_setSession()) by 
  saving the tokens, expiration, and profile to local storage and calling setLoggedIn() so that any components in the app are informed 
  that the user is now authenticated.
* Finally, we'll implement the renewToken() method, which uses the Auth0 checkSession() method to request a fresh access token from 
  Auth0 if the user's authentication session is still active. If there is no session active, we won't take any action
#### STEP-05 : Provide AuthService in App Module
In order to use the AuthService methods and properties anywhere in our app, we need to add the service to the providers array in our app.module.ts:
```
import { AuthService } from './auth/auth.service';
```
#### STEP-05 : Create a Callback Component
Next we'll create a Callback component. This is where the app is redirected after authentication. This component handles the authentication information and then shows a loading message until hash parsing is completed and the Angular app redirects back to the home page. The authentication service's handleAuth() method must be called in the callback.component.ts constructor so it will run on initialization of our app:
```
// src/app/pages/callback/callback.component.ts
import { AuthService } from './auth/auth.service';
...
  constructor(private auth: AuthService) {
    // Check for authentication and handle if hash present
    auth.handleAuth();
  }
...
All we need to do in this component's template is change the text in callback.component.html to Loading..., like so:
```
<!-- src/app/pages/callback/callback.component.html -->
<div>
  Loading...
</div>
* NOTE : ADD Component to routing module: For now, let's add the component to our routing module, app-routing.module.ts:
```
// src/app/app-routing.module.ts
...
import { CallbackComponent } from './pages/callback/callback.component';

const routes: Routes = [
  ...
  {
    path: 'callback',
    component: CallbackComponent
  }
];
```
##### STEP-06 : Add Login and Logout to Header Component
Open up the header.component.ts file:
```
// src/app/header/header.component.ts
...
import { AuthService } from './../auth/auth.service';
...
export class HeaderComponent implements OnInit {
  ...
  constructor(
    ...,
    public auth: AuthService) { }
  ...
}
```
Now let's add login, logout, and a user greeting to the header.component.html template:
```
<!-- src/app/header/header.component.html -->
<header id="header" class="header">
  <div class="header-page bg-primary">
    ...
    <div class="header-page-authStatus">
      <span *ngIf="auth.loggingIn">Logging in...</span>
      <ng-template [ngIf]="!auth.loggingIn">
        <a *ngIf="!auth.loggedIn" (click)="auth.login()">Log In</a>
        <span *ngIf="auth.loggedIn && auth.userProfile">
          {{ auth.userProfile.name }}
          <span class="divider">|</span>
          <a (click)="auth.logout()">Log Out</a>
        </span>
      </ng-template>
    </div>
  ...
```


* https://dragonprogrammer.com/securing-angular-auth0/
```
------------------------------------------------------------------------------------------------------------------------------------


# Auth0 Angular Samples

[![CircleCI](https://circleci.com/gh/auth0-samples/auth0-angular-samples.svg?style=svg)](https://circleci.com/gh/auth0-samples/auth0-angular-samples)

These samples demonstrate how to add authentication to an Angular application with Auth0, using [auth0-spa-js](https://github.com/auth0/auth0-spa-js). Each folder contains a distinct application so that various Auth0 features can be viewed in isolation. You can read about these examples in our [Angular Quickstart](https://auth0.com/docs/quickstart/spa/angular2).

Read the [full tutorials on Auth0.com](https://auth0.com/docs/quickstart/spa/angular2).

## Embedded Integration Samples

These samples use Auth0's [universal login page](https://auth0.com/docs/hosted-pages/login) which offers the fastest, most secure, and most feature-rich way to add authentication to your app.

## What is Auth0?

Auth0 helps you to:

- Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, among others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
- Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
- Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
- Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
- Analytics of how, when and where users are logging in.
- Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free Auth0 account

1. Go to [Auth0](https://auth0.com/signup) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](https://auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.
