<h1 align="center">useAuth – the simplest way to add authentication to your React app</h1>
<p>
  <img alt="Version" src="https://img.shields.io/badge/version-0.1.0-blue.svg?cacheSeconds=2592000" />
  <a href="https://github.com/Swizec/useAuth/blob/master/LICENSE">
    <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg" target="_blank" />
  </a>
</p>

> The simplest way to add authentication to your React app. Handles everything for you. Users, login forms, redirects, sharing state between components. Everything

![](https://s3.amazonaws.com/techletter.app/screenshot-1565368397655.png)

## How to use useAuth

[`useAuth`](https://github.com/Swizec/useAuth) is designed to be quick to setup. You'll need an Auth0 account with an app domain and client id.

### 1. Install the hook

```
$ yarn add react-use-auth
```

Downloads from npm, adds to your package.json, etc. You can use `npm` as well.

### 2. Set up AuthProvider

useAuth uses an `AuthProvider` component to configure the Auth0 client and share state between components. It's using React context with a reducer behind the scenes, but that's an implementation detail.

I recommend adding this around your root component. In Gatsby that's done in `gatsby-browser.js` and `gatsby-ssr.js`. Yes `useAuth` is built so it doesn't break server-side rendering. ✌️

But of course server-side "you" will always be logged out.

```javascript
// gatsby-browser.js

import React from "react";
import { navigate } from "gatsby";

import { AuthProvider } from "react-use-auth";

export const wrapRootElement = ({ element }) => (
    <AuthProvider
        navigate={navigate}
        auth0_domain="useauth.auth0.com"
        auth0_client_id="GjWNFNOHq1ino7lQNJBwEywa1aYtbIzh"
    >
        {element}
    </AuthProvider>
);
```

`<AuthProvider>` creates a context, sets up a state reducer, initializes an Auth0 client and so on. Everything you need for authentication to work in your whole app :)

The API takes a couple config options:

1. `navigate` – your navigation function, used for redirects. I've tested with Gatsby, but anything should work
2. `auth0_domain` – from your Auth0 app
3. `auth0_client_id` – from your Auth0 app
4. `auth0_params` – an object that lets you overwrite any of the default Auth0 client parameters

_PS: even though Auth doesn't do anything server-side, useAuth will throw errors during build, if its context doesn't exist_

#### Default Auth0 params

By default `useAuth`'s Auth0 client uses these params:

```javascript
const params = {
    domain: auth0_domain,
    clientID: auth0_client_id,
    redirectUri: `${callback_domain}/auth0_callback`,
    audience: `https://${auth0_domain}/api/v2/`,
    responseType: "token id_token",
    scope: "openid profile email"
};
```

`domain` and `clientID` come from your props.

`redirectUri` is set to use the `auth0_callback` page on the current domain. Auth0 redirects here after users login so you can set cookies and stuff. `useAuth` will handle this for you ✌️

`audience` is set to use api/v2. I know this is necessary but honestly have been copypasting it through several of my projects.

`responseType` same here. I copy paste this from old projects so I figured it's a good default.

`scope` you need `openid` for social logins and to be able to fetch user profiles after authentication. Profile and Email too. You can add more via the `auth0_params` override.

### 3. Create the callback page

Auth0 and most other authentication providers use OAuth. That requires redirecting your user to _their_ login form. After login, the provider redirects the user back to _your_ app.

Any way of creating React pages should work, here's what I use for Gatsby.

```javascript
// src/pages/auth0_callback

import React, { useEffect } from "react";

import { useAuth } from "react-use-auth";
import Layout from "../components/layout";

const Auth0CallbackPage = () => {
    const { handleAuthentication } = useAuth();
    useEffect(() => {
        handleAuthentication();
    }, []);

    return (
        <Layout>
            <h1>
                This is the auth callback page, you should be redirected
                immediately.
            </h1>
        </Layout>
    );
};

export default Auth0CallbackPage;
```

The goal is to load a page, briefly show some text, and run the `handleAuthentication` method from `useAuth` on page load.

That method will create a cookie in local storage with your user's information and redirect back to the homepage by default.

To redirect to a route other than the homepage after the user is logged in, supply the `handleAuthentication` function an Object Literal with the `postLoginRoute` key and an associated route value. For example, to route to `/account`, call `handleAuthentication` as follows:

```handleAuthentication({ postLoginRoute: "/account" })```


**_PS: Make sure you add `<domain>/auth0_callback` as a valid callback URL in your Auth0 config_**

### 4. Enjoy useAuth

[![](https://i.imgur.com/KunEemN.gif)](https://gatsby-useauth-example.now.sh)

You're ready to use `useAuth` for authentication in your React app.

Here's a login button for example:

```javascript
// src/pages/index.js

const Login = () => {
    const { isAuthenticated, login, logout } = useAuth();

    if (isAuthenticated()) {
        return <Button onClick={logout}>Logout</Button>;
    } else {
        return <Button onClick={login}>Login</Button>;
    }
};
```

`isAuthenticated` is a method that checks if the user's cookie is still valid. `login` and `logout` trigger their respective actions.

You can even say hello to your users

```javascript
// src/pages/index.js

const IndexPage = () => {
  const { isAuthenticated, user } = useAuth()

  return (
    <Layout>
      <SEO title="Home" />
      <h1>Hi {isAuthenticated() ? user.name : "people"}</h1>
```

Check `isAuthenticated` then use the user object. Simple as that.

## Checklist for configuring Auth0

There's a couple of required configurations you need to make in Auth0 to make useAuth run smoothly.

**Callback URLs**

You need to allow both local development and your production app in callback URLs. It's a whitelist that tells Auth0 that your login request is coming from the right source.

![](https://i.imgur.com/xz8UK8Z.png)

**Allowed Web Origins**

useAuth avoids using local storage for secure tokens. For Auth0 to know that our `checkSession` request is coming from the right source, you need to add your URLs to allowed web origins.

![](https://i.imgur.com/w2mmHH1.png)

**Allowed logout urls**

After logging out, Auth0 redirects back to your app. Again, it needs to know you aren't up to anything shady.

![](https://i.imgur.com/S160EiI.png)

---

You can try it out here 👉 https://gatsby-useauth-example.now.sh/

## Author

👤 **Swizec Teller <swizec@swizec.com>**

-   Github: [@swizec](https://github.com/swizec)
-   Twitter: [@swizec](https://twitter.com/swizec)
-   Blog: [swizec.com/blog](https://swizec.com/blog)

👤 **Mateus Gabi Moreira <mateusgabimoreira@gmail.com>**

-   Github: [@mateusgabi](https://github.com/mateusgabi)
-   Twitter: [@uptogabi](https://twitter.com/uptogabi)

## 🤝 Contributing

Contributions, issues and feature requests are welcome!<br />Feel free to check [issues page](https://github.com/Swizec/useAuth/issues).

I am looking to support other authentication providers. Please help :)

## Show your support

Give a ⭐️ if this project helped you!

## 📝 License

Copyright © 2019 [Swizec Teller <swizec@swizec.com>](https://github.com/swizec).<br />
This project is [MIT](https://github.com/Swizec/useAuth/blob/master/LICENSE) licensed.

---

_This README was generated with ❤️ by [readme-md-generator](https://github.com/kefranabg/readme-md-generator)_
