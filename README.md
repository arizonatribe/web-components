# Attainia Web Components

This repository contains modularized JavaScript and CSS meant to be used in an Attainia web application (built using the React.js framework).

## Dependencies

These components are intended to be used in a [React](https://github.com/facebook/react) web application.

Many dependencies are used to build this library and because the components in this library are able to be used "piecemeal" (rather than as pieces you pull from one single imported module), bundling all the external dependencies into each component would result in excessively large files. Perhaps in the future some bundling will be performed but for now you can install their dependencies as needed.

This means if you import a component that uses no dependencies it will be as small a file as possible. Optionally, you can import any module from the `src/` sub-folder if you want the non-transpiled component (in which case you need to handle that step in your project's own webpack/babel build process). Because they are being provided to you as ES6 modules you can take advantage of [tree shaking](https://webpack.js.org/guides/tree-shaking/) to arrive at the smallest bundle possible.

In general you can expect to encounter a "missing dependency" error (which you solve by simply running an `npm install <some missing dependency>`) rarely if you are using a normal React/Redux application (these are the primary deps).

Some components are meant to work with [react-apollo](https://github.com/apollographql/react-apollo) (such as the `AuthProvider`) and some components that collect user input are meant to work with [redux-form](https://github.com/erikras/redux-form) and [validatorjs](https://github.com/skaterdav85/validatorjs). Also, this library opts for [ramda](https://www.npmjs.com/package/ramda) [rather than lodash/underscore](https://www.youtube.com/watch?v=ixbJrJTOnF8) for many of the low-level JavaScript tasks.  

These components may or may not implement one or many of the following dependencies (most especially you'll find these used in the `.container` component wrappers):
* [redux](https://github.com/reactjs/redux) - Unidirectional data flow in a React application
* [react-redux](https://github.com/reactjs/react-redux) - Bridges React to Redux
* [react-apollo](https://github.com/apollographql/react-apollo) - Data provider to wrap around components that can run or subscribe to server queries
* [redux-form](https://github.com/erikras/redux-form) - HTML Form helpers and validation that flow into a Redux store
* [validatorjs](https://github.com/skaterdav85/validatorjs) - Build validation objects that Redux-Form can easily use
* [styled-components](https://github.com/styled-components/styled-components) - Possibly the first truly elegant marriage of CSS and JavaScript that can (potentially) please developers who fall on either side of the CSS or JS development
* [subscriptions-transport-ws](https://github.com/apollographql/subscriptions-transport-ws) - Used by any of these components that leverage GraphQL subscriptions

Additionally, certain low-level libraries are also servicing these components:
* [Ramda](https://www.npmjs.com/package/ramda) - Utils library (similar to Lodash) but more properly geared towards functional programming paradigms
* [UUID](https://github.com/kelektiv/node-uuid) - Generates guids

## Installing

```bash
npm install attainia-web-components
```

## Usage 

Many components in this library can be used in two ways. While the actual component is defined in just the JavaScript file after which it is named, there are supplementary files that expand on its functionality which you can choose to use as well.

As an example, the "Login" component consists of a `Login.js`, a `Login.containerjs`. The latter is a component that wraps `Login.js`, mapping properties from a Redux store and wrapping a Redux dispatcher around an action creator object, injecting all of those as properties into the actual `Login.js` component itself. So you are free to import _only_ the `Login.js` _if_ you take care of setting and injecting those properties into the component yourself. However if you choose to use the component from the higher order `Login.container.js` component, you need to integrate the Login components reducer into your application's reducers.

To add the reducer for the auth components into your application, just import the `reducer.js` from this library's `auth/` folder into your `combineReducers()` method. In a typical application you might have a `reducers.js` above your `components/` folder:

```javascript
import {combineReducers} from 'redux';
import {reducer as formReducer} from 'redux-form';

/* reducer for the `auth` component(s) from attainia-web-componets library */
import auth from 'attainia-web-components/auth/reducer';

/* your local reducers */
import resources from './components/resources/reducer';

/* combine your local reducers with the reducer(s) for the attain web components */
export default combineReducers({
    auth,
    resources,
    form: formReducer
});
```

Now, your `Login.container.js` will have access to the `auth` section of your application's Redux store, which is where the functionality specific to these components take place.

### Importing JS Components

The React web components in this repository are imported just like any normal local web component:

```javascript
import {Conditional} from 'attainia-web-components/common/Conditional';

export default (props) =>
    <Conditional condition={props.isLoggedIn}>
        <header>
            <h1>{`Welcome ${props.name}!`}</h1>
        </header>
    </Conditional>;
```

In addition to the basic web component files are stylesheets, types, action creators, container components (which wrap the components themselves and inject properties from the Redux store), and local stores/reducers. So you are free to use the naked component itself and inject the required properties it defines, OR you can use the _container_ component if you plan on letting the component use its own store and follow pre-configured behavior (defined in the action creators and reducers).

Using any container component will then work seamlessly, just import the `.container.js` component, _not_ the naked component itself (which you would need to configure yourself if you wanted more control).

```javascript
import React from 'react';
import {Provider} from 'react-redux';

/* attainia web component */
import Login from 'attainia-web-components/auth/Login.container';

/* the Redux store */
import store from './store';

/* Local component */
import ResourcesList from './components/resources/ResourcesList.container';

export default (
    <Provider store={store}>
        <Login>
            <ResourcesList />
        </Login>
    </Provider>
);
```

## Configuring a Theme

It is best to follow official styled-components guidelines for themeing, but you can leverage many `<ThemeProvider />` components or just one that wraps the entire React application (usually the same place you create your Redux `<Provider />`). If you _are_ using the [styled-components ThemeProvider](https://www.styled-components.com/docs/advanced#theming) you can pass the theme into it like this:

```javascript
<ThemeProvider theme={theme.catalog}>
   // All your application components which need to use the theme
</ThemeProvider>
```

Any of the children components to `<ThemeProvider />` in your application can then access the theme:

```javascript
import styled from 'styled-components'

export default styled.button`
    border-radius: 5px;
    background-color: ${props => props.theme.colors.primary.default || 'black'}
    color: ${props => props.theme.colors.secondary.default || 'white'}
`
```

Many/most of the components in this library attempt to parse themeing constants from the `theme` object passed in via the React `context`. The structure of the theme object which many of these components are expecting can be defined like this:

```javascript
{
  // simple svg icons who require only an array of coordinates to be mapped to a <g><path d="path1" /><path d="path2" /> ...</g>
  icons: {
    // Always set a primary icon
    primary: {
      paths: [
        "M56 403 ..."
      ]
    },
    delete: {
      paths: [
        "M37 47403 ..."
      ]
    },
    edit: {
      paths: [
        "M37 47403 ...",
        "M47 238 3834..."
      ]
    }
  },
  fonts: {
    fontSize: "12px",
    fontFamily: "Lato, Helvetica, sans-serif"
    // define any other font related theme-wide styles in camelcased equivalents of the native CSS properties
    ...
  },
  breakpoints: {
    phone: "screen and (min-width: 544px)",
    tablet: "screen and (min-width: 768px)",
    desktop: "screen and (min-width: 992px)",
    largeDesktop: "screen and (min-width: 1200px)"
  },
  colors: {
    primary: {
      default: "#E10600", // set a "default" value in case of simple themes OR if a non-existent property is called
      red: {
        lt: "#F0887D",
        md: "#E10600",
        dk: "#FF0700"
      },
      // define any other necessary blue, green, orange, purple, yellow, or gray ranges
      ...
    },
    secondary: {
      default: "#1b6595", // set a "default" value in case of simple themes OR if a non-existent property is called
      blue: {
        lt: "#227fbb",
        md: "#1b6595",
        dk: "#1F74B2"
      },
      // define any other red, green, orange, purple, yellow, or gray ranges
      ...
    },
    status: {
      error: "#E10600",
      warning: "#FFF59D",
      ok: "#64DD17"
    },
    grayscale: {
      white: "", // defaults to #ffffff
      black: "", // defaults to #000000
      // Always set a lt, md, and dk
      lt: "",
      md: "",
      dk: "",
      // If necessary, set an entire scale
      100: "",
      200: "",
      300: "",
      400: "",
      500: "",
      600: "",
      700: "",
      800: "",
      900: ""
    }
  }
}
```

## Component Table of Contents

Auth Components:

* [AuthProvider](#auth-provider)
* [AuthError](#auth-error)
* [Login](#login)
* [Logout](#logout)
* [PasswordHelp](#password-help)
* [Registration](#user-registration)
* [RegisterApplication](#app-registration)

Common Components:

* [Button](#button)
* [ErrorMessage](#error-message)
* [FieldError](#field-error)
* [Form](#form)
* [FormField](#form-field)
* [Link](#link)
* [LinkButton](#link-button)
* [Conditional](#conditional-rendering) 
* [ReduxFormField](#redux-form-field) 
* [SimpleSvgIcon](#simple-svg-icon)

### Auth Provider

This is the recommended way to use the `auth` components. Rather than grafting together the individual components yourself you can use this high-level component to wrap your application children components. This component will take care of the logic for rendering the `Login` component when un-authenticated and rendering your application's children components (make sure to pass in child components!) whenever users _are_ authenticated.

__options__:

* `baseUrl` - [`String`] The most important piece: the URL to your GraphQL server (defaults to `localhost`)
* `storage` - [`String`] Configures it to write tokens to and read them from browser storage. Current options are `session`, `local` and `none` (defaults to `local`) which leverages [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage), [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) or the in-memory Redux store.
* `onLogout` - A [`Function`] which fires whenever a Logout action occurs.
* `onLogin`  - A [`Function`] which passes you the `user` object on successful login attempts.

```javascript
// A "user" object passed into your (optional) onLogin callback
{
    id: String,
    email: String,
    is_active: Boolean,
    last_login: String,
    roles: [String],
    token: {
        access_token: String,
        expires_in: Number,
        token_type: String,
        scope: String,
        redirect_uris: String
    }
}
```

__usage__:

```javascript
import {Provider} from 'react-redux'
import AuthProvider from 'attainia-web-components/auth/AuthProvider'

export default (props) =>
    <Provider store={store}>
        <AuthProvider>
            <!-- render your application components that show only for authenticated users here -->
        </AuthProvider>
    </Provider>
```

### Auth Error

This wrapper around the [ErrorMessage](#error-message) component places it into an HTML element where users can close out the error by clicking on it. If you use the container component around this one, it will take advantage of Redux to pull in the error from the `auth` section of our the Redux store and to dispatch a `CLEAR_ERROR` action on click.

### Error Message

A very simple company branded (Attainia) styling around a basic `<div>` with error message text inside
### Login

Renders the Attainia user authentication component, which expects an email and password to be provided. Additionally it links to [password reset](#password-help) and [user registration](#user-registration) components.

__options__:
* `email` - [`String`] Optionally you can pre-populate the email field on the Login form
* `showRegistration` - [`Boolean`] Determines whether to display the Registration link (defaults to `false`)

### Logout

Renders a Link which (on clicked) will log out the user

### Password Help

Collects a given user's registered email address, to trigger the password reset process defined in your application.

### User Registration

New users can register with your application using this form. It currently provides name, email and password fields.

### App Registration

Allows administrative users to register an application with an OAuth provider back-end, which should return a client Id and client secret that the new application can use in future interaction with the OAuth provider.

### Conditional Rendering

Based on [mathieuancelin](https://github.com/mathieuancelin) npm module [react-conditional-render](https://www.npmjs.com/package/react-conditional-render), however its lack of attention and upkeep has caused some problems with linting and module bundling tools. Additionally, some functionality has been added here to allow its `condition` property to evaluate a greater range of "truthy" values.

In addition to the standard use:

```javascript
import {Conditional} from 'attainia-web-components/common/Conditional';

export default (props) =>
    <Conditional condition={props.isLoggedIn}>
        <header>
            <h1>{`Welcome ${props.name}!`}</h1>
        </header>
    </Conditional>;
```

You can import the `renderConditional` function for use outside of JSX (ie, in a higher order component wrapper):

```javascript
import {connect} from 'react-redux';
import {renderConditional} from 'attainia-web-components/common/Conditional';

import ResourcesDetail from './ResourcesDetail';

/* set the `condition` property and it will render the component when evaluates to true */
const mapStateToProps = state => ({
    condition: state.auth.user.id,
    resource: state.resources.detail
});

export default renderConditional(
    connect(mapStateToProps)(ResourcesDetail)
);
```

### Button

A simple HTML button which has been styled according to company (Attainia) branding

### Field Error

An error message that attaches near the [FormField](#form-field)

### Form

A simple, small styled HTML `<form>`

### Form Field

A versatile HTML `<input />` (or `<textarea />`) field with (optional) `<label />` and field-level Error, meant to be used in an HTML form (preferrably alongside [redux-form](http://redux-form.com/)). It is driven by the `label` and `type` properties, with options to attach any standard event handler via the `handlers` property. It supports numerous `type='<input type'`, but defaults to a value of `type='text'`. Additionally, it supports `placeholder` and `label`, the latter of which will render a `<label />` element before the actual `<input />` tag (except in the case of `<input type=checkbox />`, where the label renders after). Make sure to set your `id` property if you _are_ setting a `label` property, since the label must refer to the associated `<input />` element by its unique id. 

### Link

An HTML `<a>` link which has been styled according to company (Attainia) branding

### Link Button

An HTML `<a>` link wrapping the styled Attainia Button

### Simple Svg Icon

This component parses stringified paths from the `icons` object passed into your styled-components `<ThemeProvider theme={theme} />`. Just set the `icon` property and it will look for a `paths` array inside the matching icon. _Note_ this component handles simple SVGs and renders as

```
<g><path d={...} /><path d={...} /> ... </g>
```

Therefore it is not meant for complex SVGs with the myriad of other tags that can go inside of a `<svg></svg>` wrapper.

The way to use it is like so:

```javascript
<SimpleSvgIcon icon="notification" fill="crimson" width="10" height="10" />
```

### Redux Form Field

A wrapper around [FormField](#form-field), for use _specifically_ in [redux-forms](http://redux-form.com/).
