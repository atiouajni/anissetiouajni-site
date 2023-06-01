---
title: "Secure a nodeJS application with Keycloak (OIDC protocol)"
date: 2023-05-26T18:01:42+01:00
type: 'post'
categories: ['security']
tags: ['nodejs', 'keycloak', 'passeport', 'oidc']
---
Integrating Keycloak, an Identity and Access Management solution, with a Node.js application can be a powerful combination for securing your application's resources and managing user authentication. However, finding reliable and up-to-date content and tutorials that provide a step-by-step guide specifically tailored to Node.js and Keycloak integration can be challenging.

In this tutorial, I aim to address this issue by providing you with a comprehensive guide to integrating Keycloak into your Node.js application. We'll walk you through the process, from configuring Keycloak with a pre-existing realm to starting your Node.js server with Keycloak integration. By following this tutorial, you'll be able to leverage the capabilities of Keycloak to enhance the security and authentication aspects of your Node.js application.

Let's get started with the Keycloak configuration and then proceed to launch your Node.js application with Keycloak seamlessly.We'll start with setting up Keycloak by importing a pre-existing realm, and then we'll start the Node.js server.

You will find all the code in my repository [@ Github](https://github.com/atiouajni/nodejs-oidc-keycloak)

**Keycloak Integration Screens: Login, User Page, and Profile**

Once you have integrated Keycloak into your Node.js application, we'll have access to several screens that enhance the user experience and provide secure access to the application's resources. Let's take a look at the key screens involved in the Keycloak integration:

1. **Login Page**: The login page is the initial screen where users are prompted to enter their credentials to authenticate themselves. Keycloak handles the authentication process and verifies the user's identity against the configured realm.

![Login page](/img/2023-05-26/login-page.png)

2. **User Page**: After successful login, users are redirected to a protected user page where they can access secured resources within Node.js application. This page can be customized to display user-specific information or provide access to specific features based on user roles and permissions defined in Keycloak.

![User page](/img/2023-05-26/user-page.png)

3. **Profile Screen**: The profile screen allows users to view profile information. It displays attributes associated with the user, such as name, email address, and other custom attributes configured in Keycloak. Users can update their profile information or perform actions related to their account.

![Profile page](/img/2023-05-26/profile-page.png)

In the following steps, I will guide you through the configuration and setup required to utilize these screens effectively.

**Step 1: Keycloak Configuration**

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

**Step 2: Starting Node.js with Keycloak**

1. Make sure you have Node.js installed on your machine. You can download it from the official Node.js website [here](https://nodejs.org) and follow the appropriate installation instructions.

2. Open a command line or terminal and navigate to the directory of your Node.js application.

3. Rename the .env.sample file in the root directory to .env. This file will store your environment variables. You may need to adjust those environments based on your configuration.

    ```
    mv .env.sample .env
    ```

4. Open the `.env` file and provide the following configuration values for authentication:

   - `OIDC_BASE_URI`: The base URL of your Keycloak server.
   - `OIDC_CLIENT_ID`: The client ID of your application registered in Keycloak.
   - `OIDC_CLIENT_SECRET`: The client secret associated with the client ID.
   - `OIDC_REDIRECT_URI`: The redirect URI where Keycloak will redirect the user after successful authentication.

   Make sure to replace these placeholder values **if changed** with your actual Keycloak configuration details.    ```

5. Run the following command to install the necessary dependencies:

   ```
   npm install
   ```

   This will install the dependencies specified in `package.json` file.

6. Run the following command to start the Node.js server:

   ```
   npm start
   ```

   This will launch your Node.js application and make it accessible via a web browser.

7. Open a web browser and navigate to the Node.js application's URL. (localhost:3000)

8. Click on the "Login" button or link to access the login page.

9. Use the pre-configured user credentials to log in. For example:

    Username: user

    Password: password

Congratulations! You have now started your Node.js application with Keycloak. Make sure Keycloak is up and running and properly configured for authentication to work as intended. You can further customize your Node.js application based on the specific needs of your project.


**NB :** This repository is a customization of [OneLogin repository](https://github.com/onelogin/onelogin-oidc-node).
