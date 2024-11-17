---
title: "Firebase functions Error ID: 7485c5b6"
date: 2022-05-22T18:05:47+03:00
draft: false
type: post
categories: [development, k8s, kubernetes]
tags: [hugo,content,static site generator]
---

> This post is WIP.

# Introduction

I've been on a project that is using Firebase to deploy GCloud Functions.

Firebase Functions (from now on GCloud Functions) are "Serverless Functions" that are deployed as part of a Firebase Application.

Their main benefit for out use-case, is that the Firebase SDK handles authentication & authorization OOTB.

Additionally, the project benefits quite well from the NoSQL implementation for Firestore.

The experience is nice, with local emulators provided by google.

The layout is pretty standard:

```shell
.
├── README.md
├── app
│   ├── index.html
│   ├── public
│   ├── src
│   ├── tsconfig.json
│   ├── tsconfig.node.json
│   └── vite.config.ts
├── firebase.json
├── firestore.rules
├── functions
│   ├── lib
│   ├── package-lock.json
│   ├── package.json
│   ├── src
│   ├── tsconfig.dev.json
│   └── tsconfig.json
├── package-lock.json
├── package.json
├── public
    ├── builders.html
    ├── company.html
    ├── discover
    ├── index.html
    ├── mono.html
    ├── ranch.html
    ├── static
    ├── suppliers.html
    └── sustainability.html
```

Is this this the best layout out there?

Probably not, but it get's the job done.

The technological stack is simple:

* Typescript - Type checking for NodeJS
* React - Because React.. (UI..)
* Firebase - Simple to get started with
* Vite - Quick iteration build system
* EsBuild - Bundler for Cloud Functions.

# The error

Trying to run `firebase deploy --only functions` resulted with the error:

`Build failed: function.js does not exist; Error ID: 7485c5b6`

Googling the error didn't do any good, nor did any of the "modern" Chatbots provide any useful information.

> I guess it's time to roll up my sleeves and dive in.

## Looking at the configs

The `firebase.json` file had the following configuration:

```json
{
  "functions": [
    {
      "source": "functions",
      "codebase": "default",
      "runtime": "nodejs18",
      "ignore": [
        "**/node_modules/**",
        "**/src/**",
        ".env",
        ".eslintrc.js",
        "tsconfig.json",
        "tsconfig.dev.json",
        "package-lock.json",
        "package.json"
      ],
      "predeploy": [
        "npm run functions:build"
      ]
    }
  ]
}
```

Seemed well enough.
Reducing Blob storage costs by ignoring (AKA not zipping) unneeded files.

## The build environment

To reduce the risk of Cloud Functions, we set a build system that takes the Typescript files and bundles them with ESBuild.

`esbuild functions/src/index.ts --target='node18' --packages='external' --platform='node' --bundle  --outfile='functions/lib/index.js`

> Why do it this way?

Because I wanted to get the job done, and didn't want to spend time breaking each function to it's own package or following this guide: https://firebase.google.com/docs/functions/organize-functions?gen=1st

A single file works fine and is only 14KB for all the functions.

Building the single `index.js` file generates the code correctly and exports the functions fine.

To be more precise, the functions are **DEFINED** and created in GCloud, but fail to run.

> Why is this so weird?

Because if a single file defines the functions, and contains the code to run the functions, why does the deployment fail, but the functions names and GCloud functions get created anyway?!

## Understanding GCloud Functions

Fist I had to understand how GCloud Functions work.

They are very similar to AWS Lambdas, so I went searching for the "entryfile" and entrypoint.

To my surprise, there was no entry file, just function entry point.

You need to start digging into Firebase to realize that Firestore Functions are just fancy GCloud Function wrappers.

> Why is that important?!

Because all failure can be traced into GCloud function logs.

Going back to basics, let's read the first paragraph of [GCloud Functions Source Directory](https://cloud.google.com/functions/docs/writing#directory-structure)


> By default, Cloud Functions attempts to load source code from a file named index.js at the root of your function directory. To specify a different main source file, use the main field in your package.json file.

Let's have a look:

```json
{
  "name": "functions",
  "main": "lib/index.js",
  "private": true,
  "engines": {
    "node": "18"
  },
  "dependencies": {
    "axios": "^1.4.0",
    "axios-retry": "^3.5.0",
    "firebase-admin": "^11.9.0",
    "firebase-functions": "^4.4.1",
    "limiter": "^2.1.0"
  }
}
```

Looks OK!

Besides, why would `firebase deploy` be able to read the correct file, but not execute at runtime?

*Well, that's the catch.*

# Solving the problem

Remember that small `firebase.json` file from earlier?

Well let's look at it again:

```json
{
  "functions": [
    {
      "source": "functions",
      "codebase": "default",
      "runtime": "nodejs18",
      "ignore": [
        "**/node_modules/**",
        "**/src/**",
        ".env",
        ".eslintrc.js",
        "tsconfig.json",
        "tsconfig.dev.json",
        "package-lock.json",
        "package.json"
      ],
      "predeploy": [
        "npm run functions:build"
      ]
    }
  ]
}
```

If code is working at deploytime, but not in runtime, we are probably missing a file in the final artifact (zip file) generated by firebase.

```json
{
  "functions": [
    {
      "ignore": [
        ...
        "package-lock.json", <---
        "package.json"  <---
      ],
  ]
}
```

Seems like in the quest for savings, the very important files are missing from the final zip artifact!

Meaning, when the Cloudfunction tried executing, it didn't know where `main` was, because `package.json` didn't exists.

That means the GCloud Functions defaults to look for `function.js`.

The moment these to files were'nt ignored anymore, life has become happy gain, and deployments have gone back to life.


# Final Thoughts

Simplicity is an abstraction with a cost, and that cost may manifest in ways you aren't expecting.

Here the cost is negligible, a few bytes in a zip file.

However the time to debug this issue, and to track the root cause wasn't negligible at all.

Documentation should reflect the requirements of a file.

If a Zip file must contain certain elements, validation should be put in place to ensure that files adhere to their schema.

This is not to bash on GCloud specifically, but a lesson for us developers to identify and provide a great experience if we require hard requirements from a system.
