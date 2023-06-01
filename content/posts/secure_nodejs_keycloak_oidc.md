---
title: "Secure a nodeJS application with Keycloak (OIDC protocol)"
date: 2023-05-26T18:01:42+01:00
type: 'post'
categories: ['security']
tags: ['nodejs', 'keycloak', 'passeport', 'oidc']
---
Integrating Keycloak, an Identity and Access Management solution, with a NodeJS application can be a powerful combination for securing your application's resources and managing user authentication. However, finding reliable and up-to-date content and tutorials that provide a step-by-step guide specifically tailored to NodeJS and Keycloak integration can be challenging.

In this tutorial, I aim to address this issue by providing you with a comprehensive guide to integrating Keycloak into your NodeJS application. I'll walk you through the process, from configuring Keycloak with a pre-existing realm to starting your NodeJS server with Keycloak integration. By following this tutorial, you'll be able to leverage the capabilities of Keycloak to enhance the security and authentication aspects of your NodeJS application.

The NodeJS application is an [Express.js](https://expressjs.com/) app that uses [Passport.js](http://www.passportjs.org/) and the [Passport-OpenIdConnect](https://github.com/jaredhanson/passport-openidconnect) module for managing user authentication.

Let's get started with the Keycloak configuration and then proceed to launch our NodeJS application with Keycloak seamlessly. We'll start with setting up Keycloak by importing a pre-existing realm, and then we'll start the NodeJS server.

You will find all the code in my repository [@ Github](https://github.com/atiouajni/nodejs-oidc-keycloak)

## Keycloak Integration Screens: Login, User Page, and Profile

First, let's see the final result of this integration. We will have access to several screens that enhance the user experience and provide secure access to the application's resources. Let's take a look at the key screens involved in the Keycloak integration:

1. **Login Page**: The login page is the initial screen where users are prompted to enter their credentials to authenticate themselves. Keycloak handles the authentication process and verifies the user's identity against the configured realm.

![Login page](/img/2023-05-26/login-page.png)

2. **User Page**: After successful login, users are redirected to a protected user page where they can access secured resources within NodeJS application. This page can be customized to display user-specific information or provide access to specific features based on user roles and permissions defined in Keycloak.

![User page](/img/2023-05-26/user-page.png)

3. **Profile Screen**: The profile screen allows users to view profile information. It displays attributes associated with the user, such as name, email address, and other custom attributes configured in Keycloak.

![Profile page](/img/2023-05-26/profile-page.png)

In the following steps, I will guide you through the configuration and setup required to utilize these screens effectively.

## Passport.js Integration

I will not describe the usefulness of each line of code but you should know that to have a successful integration of the `Passport-OpenIdConnect` strategy, you must apply these steps:

1. Use Passport with OpenId Connect strategy to authenticate users with Keycloak
```
var passport = require('passport')
var KeycloakLoginStrategy = require('passport-openidconnect').Strategy
```

2. Initialize Passport
```
app.use(passport.initialize());
app.use(passport.session());
```

3. Configure the OpenId Connect Strategy with credentials obtained from Keycloak
```
passport.use(new KeycloakLoginStrategy({
  issuer: baseUri,
  clientID: process.env.OIDC_CLIENT_ID,
  clientSecret: process.env.OIDC_CLIENT_SECRET,
  authorizationURL: `${baseUri}/protocol/openid-connect/auth`,
  userInfoURL: `${baseUri}/protocol/openid-connect/userinfo`,
  tokenURL: `${baseUri}/protocol/openid-connect/token`,
  callbackURL: process.env.OIDC_REDIRECT_URI,
  passReqToCallback: true
}));
```

4. Add serializeUser and deserializeUser functions
```
passport.serializeUser(function(user, done) {
  done(null, user);
});

passport.deserializeUser(function(user, done) {
  done(null, user);
});
```
To maintain a login session, Passport serializes and deserializes user information to and from the session. These two functions are used to store and retrieve the authenticated user from Express.js session (express-session library).

5. Initiates an authentication request with Keycloak
```
app.get('/login', passport.authenticate('openidconnect', {
  successReturnToOrRedirect: "/",
  scope: 'profile'
}));
```
6. Only allow authenticated users to access a secured route
```
function checkAuthentication(req,res,next){
  if(req.isAuthenticated()){
      next();
  } else{
      res.redirect("/");
  }
} 

app.use('/users', checkAuthentication, users);
```

## Keycloak Configuration

1. Make sure you have Keycloak installed and running on your machine. You can download it from the official Keycloak website [here](https://www.keycloak.org/downloads.html) and follow the appropriate installation instructions.

Deploy Keycloak as a container using podman :

```
podman run --name keyclock -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin -d -p 8181:8080 quay.io/keycloak/keycloak:21.0.2 start-dev
``` 
> **Warning**
> Leaving the podman VM running will cause a time drift pretty quickly, so be carefull. Restarting podman machine resolve the issue.

2. Access the Keycloak administration interface by opening your browser and navigating to the following URL: `http://localhost:8181/admin/master/console/`. You may need to adjust the url based on your configuration.

3. Log in to the administration interface using your administrator credentials.

4. Create a realm by clicking on "Add Realm".

5. Import a pre-existing realm by clicking on "Browse" and select the JSON file `nodejs-example-realm.json`.

## Starting NodeJS with Keycloak

1. Make sure you have NodeJS installed on your machine. You can download it from the official NodeJS website [here](https://nodejs.org) and follow the appropriate installation instructions.

2. Open a command line or terminal and navigate to the directory of your NodeJS application.

3. Rename the .env.sample file in the root directory to .env. This file will store your environment variables. You may need to adjust those environments based on your configuration.

    ```
    mv .env.sample .env
    ```

4. Open the `.env` file and provide the following configuration values for authentication:

   - `OIDC_BASE_URI`: The base URL of your Keycloak server.
   - `OIDC_CLIENT_ID`: The client ID of your application registered in Keycloak.
   - `OIDC_CLIENT_SECRET`: The client secret associated with the client ID.
   - `OIDC_REDIRECT_URI`: The redirect URI where Keycloak will redirect the user after successful authentication.

   Make sure to replace these placeholder values **if changed** with your actual Keycloak configuration details.
   
5. Run the following command to install the necessary dependencies:

   ```
   npm install
   ```

   This will install the dependencies specified in `package.json` file.

6. Run the following command to start the NodeJS server:

   ```
   npm start
   ```

   This will launch your NodeJS application and make it accessible via a web browser.

7. Open a web browser and navigate to the NodeJS application's URL. (localhost:3000)

8. Click on the "Login" button or link to access the login page.

9. Use the pre-configured user credentials to log in. For example:

    Username: user

    Password: password

Congratulations! You have now started your secured NodeJS application. Make sure Keycloak is up and running and properly configured for authentication to work as intended. You can further customize your NodeJS application based on the specific needs of your project. Thanks to Passport-OpenIdConnect, the integration of the OIDC protocol becomes much easier. 

