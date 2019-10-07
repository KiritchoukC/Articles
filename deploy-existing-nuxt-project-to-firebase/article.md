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

Let's head on [Firebase console](https://console.firebase.google.com/) and click on **Add project**

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

Our project will be decomposed into 3 directories

- **src** : This is where our development files sits
- **functions** : This is where our SSR function will be
- **public** : This directory will be used by Firebase hosting

Let's start with **src**. Move all the nuxt **directories** to the **src** directory.
Only the directories !

Mine were:

- assets
- components
- layouts
- middleware
- pages
- plugins
- server
- static
- store

Leave the config files at the root (package.json, nuxt.config.js, ...)

**functions** and **public** directories will be generated automatically so let's continue.

## Update Nuxt config

## SSR function

## Automate all the things

## Conclusion
