# How to deploy Nuxt on firebase

## Introduction

I already have a working website using Nuxt and SSR so why would I move everything to Firebase ?

> SSR stands for server side rendering, you can find more informations here [Understanding Server Side Rendering](https://dev.to/christopherkade/understanding-server-side-rendering-3lk5)

There's so many reasons !
To list a few...

### Price

**Current solution**: I have to pay every month for a private server

**Firebase**: Well, for my needs, it's free.

### Configuration

**Current solution**: I have to configure everything myself. Docker containers, https, nginx reverse proxy, ...

**Firebase**: Everything you need is already done. Logging, analytics, https, custom domain, ...

### Update

**Current solution**: A change in my website ? here's the steps

- Push changes to git
- Hook on docker hub get triggered and build the container (10-15 min)
- Connect on server (1 min)
- pull the latest container version (1 min)
- find the right folder where the docker-compose.yaml is and update it (2 min)

I know I could've automated things a bit more but still...

**Firebase**: Steps

- type _firebase deploy_ in terminal (1-2 min)
- done... changes are live

You're hooked ? Obviously you are. Let me help you get it running.

## Setup the Firebase project

### Create your Firebase account

You want to use Firebase ? Well you need to [create your account](https://firebase.google.com/) first.

Done ? We can now create a new project

### Create a Firebase project

Let's head over to [Firebase console](https://console.firebase.google.com/) and click on **Add project**

Set your **project name**

Click on **Continue**

**Uncheck** Google analytics for now click on **Add Firebase**

Wait for the project initialization and click on **continue**

### Install Firebase CLI

Now with the help of NPM, we will install the firebase tools on our computer.

Simply enter this command on your favorite terminal

```bash
npm i -g firebase-tools
```

Afterwards, you should be able to login with this command

```bash
firebase login
```

A browser window will pop up and allow you to login with your google account

Alright, initial Firebase setup is done...

Before adding firebase to our project, we need to update our application project structure

## Project Structure

> I'm supposing you already have a nuxt project.
>
> If not, head over to [Nuxt website](https://nuxtjs.org/guide/installation) to create a new app.

Our project will be decomposed into 3 directories

- **src** : This is where our development files sits
- **functions** : This is where our SSR function will be
- **public** : This directory will hold the files that will be served by Firebase hosting

We won't take care of the **functions** and **public** directories. It will be generated automatically.

So create the **src** directory and move all the nuxt **directories** into it.
Only the directories, leave the configuration files at the root

You should have something like the structure below

![folder-structure](https://i.imgur.com/stoa15K.png 'Folder structure')

The app is broken now ! Let's fix it by updating the nuxt config

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

Replace _dev_ and _start_ scripts with these.

> I just prefixed the path with "src"

```json
    "dev": "cross-env NODE_ENV=development nodemon src/server/index.js --watch server",
    "start": "cross-env NODE_ENV=production node src/server/index.js",
```

You should now be able to start your application by typing **yarn dev** or **npm run dev**

> Notice that the functions directory has been created with the nuxt files in it.

## Add Firebase to the project

Like Git or NPM, Firebase CLI has its _init_ command to get everything you need quickly.

Launch the command

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

> A wild public directory appeared ! Our project structure is now complete.

We can now edit our function.

## Implement SSR function

Open the _functions/index.js_ file, remove everything and paste the code below

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

To sum it up, on each reqest, the function will pass the response and request object to the _nuxt.render(req, res)_ function which will handle the app rendering.

### Updating the _function_ package.json

The function will need the same libraries as your nuxt app. Copy the package.json dependencies to the functions/package.json dependencies

At the time of writing this article, firebase supports node version 10. In functions/package.json you can update the node engine version from 8 to 10.

Here's an example of the functions/package.json of a blank nuxt project

```json
{
  "name": "functions",
  "description": "Cloud Functions for Firebase",
  "scripts": {
    "lint": "eslint .",
    "serve": "firebase serve --only functions",
    "shell": "firebase functions:shell",
    "start": "npm run shell",
    "deploy": "firebase deploy --only functions",
    "logs": "firebase functions:log"
  },
  "engines": {
    "node": "10"
  },
  "dependencies": {
    "firebase-admin": "^8.0.0",
    "firebase-functions": "^3.1.0",
    "cross-env": "^5.2.0",
    "nuxt": "^2.3.4",
    "express": "^4.16.4",
    "vuetify": "^1.3.14",
    "vuetify-loader": "^1.0.8",
    "@nuxtjs/pwa": "^2.6.0"
  },
  "devDependencies": {
    "eslint": "^5.12.0",
    "eslint-plugin-promise": "^4.0.1",
    "firebase-functions-test": "^0.1.6"
  },
  "private": true
}
```

### Updating _firebase.json_

Replace the whole file with

```json
{
  "hosting": {
    "public": "public",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "function": "nuxtssr"
      }
    ]
  }
}
```

It will redirect all the requests to the function we've made

> Using a node version above 10 can cause some issues...
> You can use **nvm** or directly install [NodeJs 10](https://nodejs.org/en/) on your computer.

## Automate all the things

### Static files

We learned earlier that static files will be held by the _public_ directory. But what are the nuxt static files ?

There will be the nuxt app itself, the result of the **nuxt build** command.

And the static files (.jpg, .ico, .png, ...) stored into the _src/static_ directory

So we'll need to move them both in the _public_ directory, let's see how...

### Step by step

Here is what we're going to automate with the scripts

1. Clean the directories in case there's already something in it
2. Build the nuxt app
3. The built app is now in the _functions_ directory. Copy the content of the _functions/.nuxt/dist/_ directory to the _public/\_nuxt_ directory
4. Copy the static files from the _src/static/_ directory to the root of the _public_ directory
5. Install the _functions_ dependencies with yarn

> The public folder should look something like this
>
> ![public-folder](https://i.imgur.com/KGLldoM.png)

These scripts will do all that for us. So kind of them.
Add these to the main package.json file.

```json
scripts: {
    "build": "nuxt build",
    "build:firebase": "yarn clean && yarn build && yarn copy && cd \"functions\" && yarn",

    "clean": "yarn clean:public && yarn clean:functions && yarn clean:static",
    "clean:functions": "rimraf \"functions/node_modules\" && rimraf \"functions/.nuxt\"",
    "clean:public": "rimraf \"public/**/*.*!(md)\" && rimraf \"public/_nuxt\"",
    "clean:static": "rimraf \"src/static/sw.js\"",

    "copy": "yarn copy:nuxt && yarn copy:static",
    "copy:nuxt": "xcopy \"functions\\.nuxt\\dist\\*\" \"public\\_nuxt\\\" /E /Y",
    "copy:static": "xcopy \"src\\static\\*\" \"public\\\" /E /Y",

    "start:firebase": "firebase serve --only functions,hosting",

    "deploy": "firebase deploy --only functions,hosting"
}
```

> I'm using Windows, you might need to tweak the scripts a bit for other OS

### Grand finale

You can now launch these commands to start your firebase application:

```bash
yarn build:firebase
yarn start:firebase
```

And to deploy:

```bash
yarn build:firebase
yarn deploy
```

## Conclusion

You now got a server rendered nuxt application on firebase... Easy huh ?

For this article, I did an example with a blank nuxt app. Here's the final project [nuxt-on-firebase example repository](https://github.com/KiritchoukC/nuxt-on-firebase-example).

PS: It was my first article ever, feel free to criticize. I'm here to learn.
You spot an error ? Shame on me ! You can correct it by doing a pull request right here [nuxt-on-firebase repository](https://github.com/KiritchoukC/Articles/tree/master/deploy-nuxt-on-firebase)
