---
title: "Convex & Clerk"
sidebar_label: "Clerk"
sidebar_position: 10
---

import UnderTheHood from "@site/docs/auth/_under_the_hood.mdx";
import ConfigTS from "!!raw-loader!@site/../private-demos/quickstarts/react-vite-ts/src/_mainClerk.tsx";
import ConfigJS from "!!raw-loader!@site/../private-demos/quickstarts/react-vite/src/_mainClerk.jsx";
import ConfigEnvTS from "!!raw-loader!@site/../private-demos/quickstarts/react-vite-ts/src/_mainClerkEnv.tsx";
import ConfigEnvJS from "!!raw-loader!@site/../private-demos/quickstarts/react-vite/src/_mainClerkEnv.jsx";
import App from "!!raw-loader!@site/../private-demos/snippets/src/clerkApp.tsx";
import Messages from "!!raw-loader!@site/../private-demos/snippets/convex/clerkMessages.ts";

[Clerk](https://clerk.com) is an authentication platform providing login via
passwords, social identity providers, one-time email or SMS access codes, and
multi-factor authentication and basic user management.

**Example:**
[TanStack Start with Convex and Clerk](https://github.com/get-convex/convex-demos/tree/main/tanstack-start-clerk)

To use Clerk with Convex you'll need to complete at least steps 1 through 7 of
this list.

If you're using Next.js see the
[Next.js setup guide](/client/react/nextjs/nextjs.mdx) after step 7.

If you're using TanStack Start see the
[TanStack Start setup guide](/client/react/tanstack-start/clerk.mdx) after
step 7.

## Get started

This guide assumes you already have a working React app with Convex. If not
follow the [Convex React Quickstart](/docs/quickstart/react.mdx) first. Then:

<StepByStep>
  <Step title="Sign up for Clerk">
    Sign up for a free Clerk account at [clerk.com/sign-up](https://dashboard.clerk.com/sign-up).

    <p style={{textAlign: 'center'}}>
      <img src="/screenshots/clerk-signup.png" alt="Sign up to Clerk" width={200} />
    </p>

  </Step>
  <Step title="Create an application in Clerk">
    Choose how you want your users to sign in.

    <p style={{textAlign: 'center'}}>
      <img src="/screenshots/clerk-createapp.png" alt="Create a Clerk application" width={200} />
    </p>

  </Step>
  <Step title="Create a JWT Template">
    In the JWT Templates section of the Clerk dashboard tap on
    _+&nbsp;New&nbsp;template_ and choose **Convex**

    Copy the _Issuer_ URL from the Issuer input field.

    Hit _Apply Changes_.

    Note: Do NOT rename the JWT token, it must be called `convex`.

    <p style={{textAlign: 'center'}}>
      <img src="/screenshots/clerk-createjwt.png" alt="Create a JWT template" width={400} />
    </p>

  </Step>
  <Step title="Create the auth config">
    In the `convex` folder create a new file <JSDialectFileName name="auth.config.ts" /> with
    the server-side configuration for validating access tokens.

    Paste in the _Issuer_ URL from the JWT template and set `applicationID` to `"convex"` (the value
    of the `"aud"` _Claims_ field).

    ```ts title="convex/auth.config.ts"
    export default {
      providers: [
        {
          domain: "https://your-issuer-url.clerk.accounts.dev/",
          applicationID: "convex",
        },
      ]
    };
    ```

  </Step>
  <Step title="Deploy your changes">
    Run `npx convex dev` to automatically sync your configuration to your backend.

    ```sh
    npx convex dev
    ```

  </Step>
  <Step title="Install clerk">
    In a new terminal window, install the Clerk React library

    ```sh
    npm install @clerk/clerk-react
    ```

  </Step>
  <Step title="Get your Clerk Publishable key">
    On the Clerk dashboard in the API Keys section copy the Publishable key

    ![Clerk publishable key setting](/screenshots/clerk-pkey.png)

  </Step>
  <Step title="Configure ConvexProviderWithClerk">
    Now replace your `ConvexProvider` with `ClerkProvider` wrapping
    `ConvexProviderWithClerk`.

    Pass the Clerk `useAuth` hook to the `ConvexProviderWithClerk`.

    Paste the Publishable key as a prop to `ClerkProvider`.


    
```typescript
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";
import { ClerkProvider, useAuth } from "@clerk/clerk-react";
import { ConvexProviderWithClerk } from "convex/react-clerk";
import { ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(import.meta.env.VITE_CONVEX_URL as string);

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ClerkProvider publishableKey="pk_test_...">
      <ConvexProviderWithClerk client={convex} useAuth={useAuth}>
        <App />
      </ConvexProviderWithClerk>
    </ClerkProvider>
  </React.StrictMode>,
);

```


  </Step>

  <Step title="Show UI based on authentication state">
    You can control which UI is shown when the user is signed in or signed out
    with the provided components from `"convex/react"` and `"@clerk/clerk-react"`.

    To get started create a shell that will let the user sign in and sign out.

    Because the `Content` component is a child of `Authenticated`, within it and
    any of its children authentication is guaranteed and Convex queries can
    require it.

    
```typescript
import { SignInButton, UserButton } from "@clerk/clerk-react";
import { Authenticated, Unauthenticated, useQuery } from "convex/react";
import { api } from "../convex/_generated/api";

function App() {
  return (
    <main>
      <Unauthenticated>
        <SignInButton />
      </Unauthenticated>
      <Authenticated>
        <UserButton />
        <Content />
      </Authenticated>
    </main>
  );
}

function Content() {
  const messages = useQuery(api.messages.getForCurrentUser);
  return <div>Authenticated content: {messages?.length}</div>;
}

export default App;

```


  </Step>

  <Step title="Use authentication state in your Convex functions">
    If the client is authenticated, you can access the information
    stored in the JWT via `ctx.auth.getUserIdentity`.

    If the client isn't authenticated, `ctx.auth.getUserIdentity` will return `null`.

    **Make sure that the component calling this query is a child of `Authenticated` from
    `"convex/react"`**, otherwise it will throw on page load.

    
```typescript
import { query } from "./_generated/server";

export const getForCurrentUser = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (identity === null) {
      throw new Error("Not authenticated");
    }
    return await ctx.db
      .query("messages")
      .filter((q) => q.eq(q.field("author"), identity.email))
      .collect();
  },
});

```


  </Step>
</StepByStep>

## Login and logout Flows

Now that you have everything set up, you can use the
[`SignInButton`](https://clerk.com/docs/components/authentication/sign-in)
component to create a login flow for your app.

The login button will open the Clerk configured login dialog:

```tsx title="src/App.tsx"
import { SignInButton } from "@clerk/clerk-react";

function App() {
  return (
    <div className="App">
      <SignInButton mode="modal" />
    </div>
  );
}
```

To enable a logout flow you can use the
[`SignOutButton`](https://clerk.com/docs/components/unstyled/sign-out-button) or
the [`UserButton`](https://clerk.com/docs/components/user/user-button) which
opens a dialog that includes a sign out button.

## Logged-in and logged-out views

Use the [`useConvexAuth()`](/api/modules/react#useconvexauth) hook instead of
Clerk's `useAuth` hook when you need to check whether the user is logged in or
not. The `useConvexAuth` hook makes sure that the browser has fetched the auth
token needed to make authenticated requests to your Convex backend, and that the
Convex backend has validated it:

```tsx title="src/App.ts"
import { useConvexAuth } from "convex/react";

function App() {
  const { isLoading, isAuthenticated } = useConvexAuth();

  return (
    <div className="App">
      {isAuthenticated ? "Logged in" : "Logged out or still loading"}
    </div>
  );
}
```

You can also use the `Authenticated`, `Unauthenticated` and `AuthLoading` helper
components instead of the similarly named Clerk components. These components use
the `useConvex` hook under the hood:

```tsx title="src/App.ts"
import { Authenticated, Unauthenticated, AuthLoading } from "convex/react";

function App() {
  return (
    <div className="App">
      <Authenticated>Logged in</Authenticated>
      <Unauthenticated>Logged out</Unauthenticated>
      <AuthLoading>Still loading</AuthLoading>
    </div>
  );
}
```

## User information in functions

See [Auth in Functions](/docs/auth/functions-auth.mdx) to learn about how to
access information about the authenticated user in your queries, mutations and
actions.

See [Storing Users in the Convex Database](/docs/auth/database-auth.mdx) to
learn about how to store user information in the Convex database.

## User information in React

You can access information about the authenticated user like their name from
Clerk's [`useUser`](https://clerk.com/docs/reference/clerk-react/useuser) hook.
See the [User doc](https://clerk.com/docs/reference/clerkjs/user) for the list
of available fields:

```tsx title="src/badge.ts"
import { useUser } from "@clerk/clerk-react";

export default function Badge() {
  const { user } = useUser();
  return <span>Logged in as {user.fullName}</span>;
}
```

## Next.js, React Native Expo, Gatsby

The `ConvexProviderWithClerk` component supports all Clerk integrations based on
the `@clerk/clerk-react` library. Replace the package from which you import the
`ClerkProvider` component and follow any additional steps in Clerk docs.

## Configuring dev and prod instances

To configure a different Clerk instance between your Convex development and
production deployments you can use environment variables configured on the
Convex dashboard.

### Configuring the backend

First, change your <JSDialectFileName name="auth.config.ts" /> file to use an
environment variable:

```ts title="convex/auth.config.ts"
export default {
  providers: [
    {
      domain: process.env.CLERK_JWT_ISSUER_DOMAIN,
      applicationID: "convex",
    },
  ],
};
```

**Development configuration**

Open the Settings for your dev deployment on the Convex
[dashboard](https://dashboard.convex.dev) and add the variables there:

<p style={{ textAlign: "center" }}>
  <img
    src="/screenshots/clerk-convex-dashboard.png"
    alt="Convex dashboard dev deployment settings"
    width={600}
  />
</p>

Now switch to the new configuration by running `npx convex dev`.

**Production configuration**

Similarly on the Convex [dashboard](https://dashboard.convex.dev) switch to your
production deployment in the left side menu and set the values for your
production Clerk instance there.

Now switch to the new configuration by running `npx convex deploy`.

### Configuring a React client

To configure your client you can use environment variables as well. The exact
name of the environment variables and the way to refer to them depends on each
client platform (Vite vs Next.js etc.), refer to our corresponding
[Quickstart](/docs/quickstarts.mdx) or the relevant documentation for the
platform you're using.

Change the props to `ClerkProvider` to take in an environment variable:


```typescript
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";
import { ClerkProvider, useAuth } from "@clerk/clerk-react";
import { ConvexProviderWithClerk } from "convex/react-clerk";
import { ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(import.meta.env.VITE_CONVEX_URL as string);

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ClerkProvider publishableKey={import.meta.env.VITE_CLERK_PUBLISHABLE_KEY}>
      <ConvexProviderWithClerk client={convex} useAuth={useAuth}>
        <App />
      </ConvexProviderWithClerk>
    </ClerkProvider>
  </React.StrictMode>,
);

```


**Development configuration**

Use the `.env.local` or `.env` file to configure your client when running
locally. The name of the environment variables file depends on each client
platform (Vite vs Next.js etc.), refer to our corresponding
[Quickstart](/docs/quickstarts.mdx) or the relevant documentation for the
platform you're using:

```py title=".env.local"
VITE_CLERK_PUBLISHABLE_KEY="pk_test_..."
```

**Production configuration**

Set the environment variable in your production environment depending on your
hosting platform. See [Hosting](/docs/production/hosting/hosting.mdx).

## Debugging authentication

If a user goes through the Clerk login flow successfully, and after being
redirected back to your page `useConvexAuth` gives `isAuthenticated: false`,
it's possible that your backend isn't correctly configured.

The <JSDialectFileName name="auth.config.ts" /> file in your `convex/` directory
contains a list of configured authentication providers. You must run
`npx convex dev` or `npx convex deploy` after adding a new provider to sync the
configuration to your backend.

For more thorough debugging steps, see
[Debugging Authentication](/docs/auth/debug.mdx).

## Under the hood

<UnderTheHood
  provider="Clerk"
  integrationProvider={<code>ConvexProviderWithClerk</code>}
  providerProvider={<code>ClerkProvider</code>}
  configProp={
    <>
      the{" "}
      <a
        href="https://clerk.com/docs/authentication/sign-in#override-ur-ls"
        target="_blank"
      >
        <code>afterSignIn</code>
      </a>{" "}
      prop
    </>
  }
/>
