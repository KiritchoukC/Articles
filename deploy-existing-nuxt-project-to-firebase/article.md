# How to host Nuxt on firebase

## Table of contents

1. Introduction
2. Setup firebase
3. Project Structure
4. Update Nuxt config
5. SSR function
6. Automate all the things
7. Conclusion

## Introduction

I already have a working website using Nuxt and SSR so why would I move everything to Firebase ?

> SSR stands for server side rendering, you can find more informations here [Understanding Server Side Rendering](https://dev.to/christopherkade/understanding-server-side-rendering-3lk5)

Well, because:

- it is hosted on a VPS, it's way too much for my need and I could save a bit of money
- The website is in a docker container. Docker is nice but it's an overkill for my website.
- Updating the website/container is too slow, you'll see later that deploying on firebase is faster and easier.
- I had to configure https myself with other docker containers. It is less secure
- And many more...

If you want an easy way to build, update and scale your website then this article can get you running.

## Setup Firebase

### Create your Firebase account

You want to use Firebase ? Well you need to [create your account](https://firebase.google.com/) first.

Done ? We can now create a new project

### Create a Firebase project

Let's head over to [Firebase console](https://console.firebase.google.com/) and click on **Add project**

Set your **project name**

Click on **Continue**

**Uncheck** Google analytics for now click on **Add Firebase**

Wait for the project initialization and click on **continue**

### Setup Firebase-tools

Now with the help of NPM, we will install the firebase tools on our computer.

Simply enter this command on your favorite terminal

```bash
npm i -g firebase-tools
```

You should be able to login with this command

```bash
firebase login
```

A browser window will pop up and allow you to login with your google account

Ok initial Firebase setup is done...

Before adding firebase to our project, we need to update our project structure

## Project Structure

> I'm moving an existing Nuxt project but if you want to start fresh, head over to [Nuxt website](https://nuxtjs.org/guide/installation) to create a new app.

Our project will be decomposed into 3 directories

- **src** : This is where our development files sits
- **functions** : This is where our SSR function will be
- **public** : This directory will be used by Firebase hosting

**functions** and **public** directories will be generated automatically.
So create the **src** directory and move all the nuxt **directories** into it.
Only the directories, leave the configuration files at the root

You should have something like the structure below

![folder-structure](https://i.imgur.com/stoa15K.png 'Folder structure')

It's broken now ! Let's fix it by updating the nuxt config

## Update Nuxt config

In nuxt.config.js, add the following lines in module.exports

```javascript
module.exports = {
[...]
  srcDir: 'src',
  buildDir: 'functions/.nuxt',
[...]
}
```

And in the build object, set extractCss to true

```javascript
module.exports = {
[...]
  build: {
    extractCSS: true,
    [...]
  }
[...]
}
```

It is still broken because npm script cannot find our entry file **server/index.js**

Let's update our package.json

replace dev an start scripts with these. I just prefixed the path with "src"

```json
    "dev": "cross-env NODE_ENV=development nodemon src/server/index.js --watch server",
    "start": "cross-env NODE_ENV=production node src/server/index.js",
```

You should now be able to start your application by typing **yarn dev** or **npm run dev**

> Notice that the functions directory has been created with the nuxt files in it.

## Add Firebase

Firebase has a simple command line for generating everything we need.

Start by typing

```bash
firebase init
```

The CLI will ask you some questions and here are the answers:

```bash
? Are you ready to proceed?
> Yes

? Which Firebase CLI features do you want to set up for this folder? Press Space to select features, then Enter to confirm your choices.
> Functions: Configure and deploy Cloud Functions,
> Hosting: Configure and deploy Firebase Hosting sites

? Please select an option:
> Use an existing project
(Select the project we created earlier)

? What language would you like to use to write Cloud Functions? (Use arrow keys)
> JavaScript

? Do you want to use ESLint to catch probable bugs and enforce style? (y/N)
> y

? Do you want to install dependencies with npm now? (Y/n)
> Y

? What do you want to use as your public directory? (public)
> public

? Configure as a single-page app (rewrite all urls to /index.html)? (y/N)
> N

```

We will now edit the generated function. Open the functions/index.js file.

Remove everything and paste the code below

```javascript
const functions = require('firebase-functions')
const { Nuxt } = require('nuxt')
const express = require('express')

const app = express()

const config = {
  dev: false
}

const nuxt = new Nuxt(config)

let isReady = false
const readyPromise = nuxt
  .ready()
  .then(() => {
    isReady = true
  })
  .catch(() => {
    process.exit(1)
  })

async function handleRequest(req, res) {
  if (!isReady) {
    await readyPromise
  }
  res.set('Cache-Control', 'public, max-age=1, s-maxage=1')
  await nuxt.render(req, res)
}

app.get('*', handleRequest)
app.use(handleRequest)
exports.nuxtssr = functions.https.onRequest(app)
```

I won't go into the details but what the function does is on request, it will pass the response and request object to the _nuxt.render(req, res)_ function which will handle the app rendering.

The function will need the same libraries as your nuxt app. Copy the package.json dependencies to the functions/package.json dependencies

At the time of writing this article, firebase supports node version 10. In functions/package.json you can update the node engine version from 8 to 10.

> If you have difficulties running the project locally, you might need to install [NodeJs 10](https://nodejs.org/en/) on your computer.

## Automate all the things

## Conclusion
