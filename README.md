# Cloud Functions for Firebase Tutorial


- [Cloud Functions for Firebase Tutorial](#cloud-functions-for-firebase-tutorial)
  - [Servers are obsolete](#servers-are-obsolete)
  - [Firebase Local Emulator Suite](#firebase-local-emulator-suite)
- [Getting Started](#getting-started)
  - [Install your app or front-end framework](#install-your-app-or-front-end-framework)
  - [Install Firebase node modules](#install-firebase-node-modules)
  - [Setting up TypeScript](#setting-up-typescript)
    - [Set up emulators](#set-up-emulators)
    - [Update npm packages and `package.json` (optional)](#update-npm-packages-and-packagejson-optional)
    - [Update `tsconfig.json` (optional)](#update-tsconfigjson-optional)
    - [Transpile TypeScript into JavaScript (NOT OPTIONAL!)](#transpile-typescript-into-javascript-not-optional)
    - [Start emulators](#start-emulators)
- [Call and trigger Cloud Functions](#call-and-trigger-cloud-functions)
  - [Callable vs. triggerable functions](#callable-vs-triggerable-functions)
  - [HTTPS callable functions](#https-callable-functions)
    - [Call your Cloud Function](#call-your-cloud-function)
    - [Data and context](#data-and-context)
    - [`httpsCallable` vs. `httpsCallableFromURL`](#httpscallable-vs-httpscallablefromurl)
      - [The emulator URL](#the-emulator-url)
      - [The Firebase Cloud URL](#the-firebase-cloud-url)
  - [Trigger your Cloud Function](#trigger-your-cloud-function)
- [Writing Cloud Functions](#writing-cloud-functions)
  - [Node.js](#nodejs)
  - [Initialize `admin`](#initialize-admin)
    - [`require` vs. \`import`](#require-vs-import)
  - [Terminate Cloud Functions with `return` or promises](#terminate-cloud-functions-with-return-or-promises)
    - [`return`](#return)
  - [Firestore get, set, add, update, delete, listDocuments](#firestore-get-set-add-update-delete-listdocuments)
    - [CREATE: `set`](#create-set)
    - [UPDATE: `update`](#update-update)
    - [READ: `get`](#read-get)
    - [DELETE: `delete`](#delete-delete)
    - [Other methods](#other-methods)
    - [Async call](#async-call)
  - [Storage](#storage)
    - [`uploadBytes` with `['rawBody']`](#uploadbytes-with-rawbody)
    - [Write to Storage from your app or front end](#write-to-storage-from-your-app-or-front-end)
- [Deploy your Cloud Function to Firebase](#deploy-your-cloud-function-to-firebase)
  - [Manage your credentials](#manage-your-credentials)
    - [Use an IAM service account](#use-an-iam-service-account)
- [Use Functions in your Firebase Console](#use-functions-in-your-firebase-console)
- [Testing Cloud Functions](#testing-cloud-functions)


This tutorial teaches how to use [Cloud Functions for Firebase](https://firebase.google.com/docs/functions) with the Firestore database, Angular, and AngularFire. You should be able to adapt this tutorial for other apps or frameworks.

I use Cloud Functions to call APIs such as Google Cloud Translation and IBM Watson Speech-to-Text. You don't want to call APIs from your front end because this exposes your API keys or other credentials. You can safely call APIs from your back end, get the data, process the data, and write the data to your databases or send the data to your front end. 

The official documentation [Explore use cases](https://firebase.google.com/docs/functions/use-cases) goes into depth on other use cases for Cloud Functions. For example, you can trigger a function when data is written to your database. You can perform intensive data processing operations or database sanitization and maintenance.

## Servers are obsolete

The "old school" way is to use a server as your back end. In the 2010s running your own server became obsolete as cloud computing services became available. Now you can run a server in the cloud or you can skip the server and just write functions that run in the cloud. Google Cloud Functions is one provider of this service.

Cloud Functions for Firebase is a "wrapper" on Google Cloud Functions that adds funcionality if you're using Firebase databases. Cloud Functions for Firebase makes it easy to write data to Firebase databases. Cloud Functions also enables you to trigger functions when data writes to your databases.

## Firebase Local Emulator Suite
There are a couple reasons not to use Cloud Functions for Firebase. First, if you write an infinite loop you could get a big bill from Google. This can happen when you trigger a function when data is written to your database, and then the function processes the data and writes new data to the database, which triggers the function again.

Another reason not to use Cloud Functions for Firebase is that deploying an updated function to the cloud takes about ten minutes. Making a change in your code and then waiting ten minutes to see the results is frustrating.

Both of these problems are solved with the Firebase Local Emulator Suite. I can update my code and see the results in a minute. The emulator is running on your laptop so there are never any charges from Google.

# Getting Started

I'll assume that you're reading the [official documentation](https://firebase.google.com/docs/functions/get-started) and I won't go into depth repeating the information. I'll provide more detail in the section on setting up TypeScript.

Create a Firebase project from your [Firebase console](https://console.firebase.google.com/u/0/).

Install [Node.js](https://nodejs.org/en/) on your computer.

Install the Firebase CLI globally.

```
npm install -g firebase-tools
```

## Install your app or front-end framework

Set up your project directory before you install Cloud Functions. For Angular:

```
ng new MyProject
```

You don't need Angular routing for this project.

If you don't want to install a framework you might want to initialize a Github repository.

Install AngularFire, if you're using Angular. 

```
cd MyProject
ng add @angular/fire
```

Set up `Firestore`, `Cloud Functions (callable)`, and `Cloud Storage`.

## Install Firebase node modules

In the project directory, install the Firebase Functions and Firebase Admin packages locally.

```
npm install firebase-functions
npm install firebase-admin
```

Initialize your project.

```
firebase login
firebase init
```

Select `Firestore`, `Functions`, `Storage`, and `Emulators`.

Select your Firebase project. Accept the defaults.

You're then asked whether you want to use JavaScript or TypeScript. If you select JavaScript, skip the next section.

## Setting up TypeScript

If you select TypeScript, the next question asks if you want to use ESLint. My choice is no, as I prefer to have Visual Studio Code do linting as I write my code. 

The last question asks about installing dependencies. Say yes.

Accept the defaults for any other questions. 

### Set up emulators

You might be asked to set up the emulators in the middle of setting up TypeScript. Select `Firestore`, `Functions`, and `Storage`.

Accept the default ports and other questions.

### Finish setting up TypeScript
Then you'll see:

```
✔  Firebase initialization complete!
```

Don't believe it!

You have another step to set up TypeScript. This is explained in [Use TypeScript for Cloud Functions](https://firebase.google.com/docs/functions/typescript). This documentation is 14 documents down from the [Get started](https://firebase.google.com/docs/functions/get-started) documentation. Read that documentation now or do the following steps.

Install TypeScript globally.

```
npm install -g typescript
```

### Update npm packages and `package.json` (optional)

Open your project in your code editor. You should have two `package.json` files, one for your app or front-end framework and one for your Cloud Functions.

```
MyProject/package.json // app or front-end framework
MyProject/functions/package.json // Cloud Functions
```

In `MyProject/functions/`, check your npm packages versions:

```
cd functions
npm outdated
```

Open `functions/package.json`. Update the version numbers of `firebase-admin`, `firebase-functions`, and `typescript`. 

The latest TypeScript version is at

```
https://www.npmjs.com/package/typescript
```

Run `npm outdated` weekly and when it shows new versions available update your npm packages:

```
npm update
```

### Update `tsconfig.json` (optional)

Open `functions/tsconfig.json`. Add

```js
"moduleResolution": "node",
```

Change `target` to `ESNext`.

### Transpile TypeScript into JavaScript (NOT OPTIONAL!)

Open `functions/src/index.ts`. Remove the comments from the default function `helloWorld`.

From the `functions` directory, run

```
npm run build
```

This transpiles TypeScript into JavaScript. It will also make several new directories and files. Check your directory structure, it should look like:

```
MyProject
├── firebase.json
├── firestore.indexes.json
├── firestore.rules
├── functions
│   ├── lib
│   │   ├── index.js
│   │   └── index.js.map
│   ├── node_modules
│   ├── package-lock.json
│   ├── package.json
│   ├── src
│   │   └── index.ts
│   └── tsconfig.json
```

You write your function in `functions/src/index.ts`. Then you run `npm run build` and TypeScript transpiles `index.ts` to JavaScript in `functions/lib/index.js`. Your functions run from `index.js`. You can see this in `package.json` in the line `"main": "lib/index.js",`. That's where your functions run from.

If you don't transpile your TypeScript into JavaScript you'll get this error message:

```
 SyntaxError: Cannot use import statement outside a module
```

## How to break TypeScript

The wrong code in `index.ts` will cause the TypeScript compiler to remove `lib/index.js` and `lib/index.js.map` and replace them with [a directory structure that you don't want](https://stackoverflow.com/questions/74831161/setdoc-write-to-firestore-from-typescript-functions-messes-up-directory-struct):

```
MyProject
├── firebase.json
├── firestore.indexes.json
├── firestore.rules
├── functions
│   ├── lib
│   │   ├── environments
│   │   |   ├── environments.js
│   │   |   ├── environments.js.map
│   │   └── functions
│   │   |   ├── src
│   │   |   |   ├── index.ts
│   │   |   |   ├── index.ts.map
│   ├── node_modules
│   ├── package-lock.json
│   ├── package.json
│   ├── src
│   │   └── index.ts
│   └── tsconfig.json
```

If you see this directory structure delete `/lib/environments` and `/lib/functions` and delete the code you wrote that broke the transpiler.

You can break the transpiler by calling Firebase modules that belong in your Angular front-end app, not in your back-end Cloud Functions:

```
import { initializeApp } from "firebase/app";
import { getFirestore, setDoc, doc } from "firebase/firestore";
const firebaseApp = initializeApp(environment.firebase);
const firestore = getFirestore(firebaseApp); //
```

### Start emulators

Gentleman, start your emulators!

In your project directory, 

```
firebase emulators:start
```

You should see

```
┌─────────────────────────────────────────────────────────────┐
│ ✔  All emulators ready! It is now safe to connect your app. │
│ i  View Emulator UI at http://127.0.0.1:4000/               │
└─────────────────────────────────────────────────────────────┘

┌───────────┬────────────────┬─────────────────────────────────┐
│ Emulator  │ Host:Port      │ View in Emulator UI             │
├───────────┼────────────────┼─────────────────────────────────┤
│ Functions │ 127.0.0.1:5001 │ http://127.0.0.1:4000/functions │
├───────────┼────────────────┼─────────────────────────────────┤
│ Firestore │ 127.0.0.1:8080 │ http://127.0.0.1:4000/firestore │
├───────────┼────────────────┼─────────────────────────────────┤
│ Storage   │ 127.0.0.1:9199 │ http://127.0.0.1:4000/storage   │
└───────────┴────────────────┴─────────────────────────────────┘
```

Now you've completed Firebase initialization! 

# Call vs. trigger Cloud Functions

Cloud Functions for Firebase can be executed in three ways:

* Call a Cloud Function from your app or front end framework such as Angular or React with an HTTPS callable function.
* Call a Cloud Function via [HTTP request](https://firebase.google.com/docs/functions/http-events) from a back end framework such as Express.
* Trigger a Cloud Function from a write to Firestore or another database.

I don't use Express so I won't talk about calling Cloud Functions with HTTP requests.

## Callable vs. triggerable functions

To execute a Cloud Function from your app or front-end framework, write a HTTPS callable Cloud Function. The code is simple, the execution is fast, and your Cloud Functions returns the resulting data to your app or front-end.

You can trigger a Cloud Function by writing to your database from your app or front-end framework but this isn't as good. It's more code, especially if you want to return data to your app or front end. And writing to your database is slower. If your app or front-end wants to get data from an API, when the data comes back your Cliud Function has to write the data to your database. Then you set up an Observer in your app or front-end to listen for changes in your database. This is a lot of code. Then when the data comes back you'll want to unsubscribe the listener. I haven't found a clean way to unscubscribe the listener when the data comes back.

If you want to use a listener for a long time, i.e., for many data changes in your database, then triggering a Cloud Function with a database write might make sense. But if your app or front-end calls a Cloud Function to get data once from an API, this isn't what listeners are good at. It's better to use a HTTPS callable Cloud Function.

If your Cloud Function triggers from events in your database, not from your app or front-end, this is where you use a triggerable Cloud Function.

## HTTPS callable functions

Copy and paste this Cloud Function in `index.ts`:

*index.js*
```js
export const upperCaseMe = functions.https.onCall((data, context) => {
    const original: string = data.text;
    const uppercase: string = original.toUpperCase();
    functions.logger.log('upperCaseMe', original, uppercase);
    return uppercase;
});
```

From your `functions` directory transpile your TypeScript:

```
npm run build
```

Make a button in your app or front-end framework. The following code is for Angular.

*app.component.html*
```html
<h3>Call cloud function</h3>
<form (ngSubmit)="callMe()">
    <input type="text" [(ngModel)]="messageText" name="message" placeholder="message" required>
    <button type="submit" value="Submit">Submit</button>
</form>
```

Import `FormsModule`:

*app.module.ts*
```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';

// Angular
import { environment } from '../environments/environment';
import { FormsModule } from '@angular/forms';

// AngularFire
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { provideFirestore, getFirestore, connectFirestoreEmulator } from '@angular/fire/firestore';
import { provideFunctions,getFunctions, connectFunctionsEmulator } from '@angular/fire/functions';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    provideFirebaseApp(() => initializeApp(environment.firebase)),
    provideFirestore(() => {
      const firestore = getFirestore();
      if (!environment.production) {
        connectFirestoreEmulator(firestore, 'localhost', 8080);
      }
      return firestore;
    }),
    provideFunctions(() => {
      const functions = getFunctions();
      if (!environment.production) {
        connectFunctionsEmulator(functions, 'localhost', 5001);
      }
      return functions;
    }),
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

*environements/environment.ts
```js
export const environment = {
  firebase: {
    projectId: 'my-projectId',
    appId: '...',
    databaseURL: 'https://my-projectId.firebaseio.com',
    storageBucket: 'my-projectId.appspot.com',
    locationId: 'us-central',
    apiKey: '...',
    authDomain: 'my-projectId.firebaseapp.com',
    messagingSenderId: '...',
  },
  production: false
};
```

*app.component.ts*
```js
import { Component, Inject } from '@angular/core';
import { Functions, httpsCallable, httpsCallableFromURL } from '@angular/fire/functions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})

export class AppComponent {
  constructor(
    @Inject(Functions) private readonly functions: Functions,
  ) {}

  messageText: string = '';

  callMe() {
    console.log("Calling Cloud Function: " + this.messageText);
    const upperCaseMe = httpsCallable(this.functions, 'upperCaseMe');
    // const upperCaseMe = httpsCallableFromURL(this.functions, 'http://127.0.0.1:5001/my-projectId/us-central1/upperCaseMe');
    upperCaseMe({ text: this.messageText })
      .then((result) => {
        console.log(result.data)
      })
      .catch((error) => {
        console.error(error);
      });;
  };
}
```

### Call your Cloud Function

Open your browser and your browser console. Enter something in the form field and click `Submit`. In your console you should see:

```
Calling Cloud Function: this is lowercase
THIS IS LOWERCASE
```

Open another browser tab for your emulator at `http://127.0.0.1:4000/functions`. Open the `Logs` tab and you should see something like:

```
18:35:25 I
function[us-central1-upperCaseMe]
Beginning execution of "upperCaseMe"
18:35:25 I
function[us-central1-upperCaseMe]
Finished "upperCaseMe" in 1.244812ms
18:35:25 I
function[us-central1-upperCaseMe]
Beginning execution of "upperCaseMe"
18:35:25 I
function[us-central1-upperCaseMe]
{
  "verifications": {
    "app": "MISSING",
    "auth": "MISSING"
  },
  "logging.googleapis.com/labels": {
    "firebase-log-type": "callable-request-verification"
  },
  "severity": "INFO",
  "message": "Callable request verification passed"
}
18:35:25 I
function[us-central1-upperCaseMe]
{
  "severity": "INFO",
  "message": "upperCaseMe this is lowercase THIS IS LOWERCASE"
}
18:35:25 I
function[us-central1-upperCaseMe]
Finished "upperCaseMe" in 3.944818ms
18:35:30 W
function[us-central1-upperCaseMe]
Your function timed out after ~60s. To configure this timeout, see
      https://firebase.google.com/docs/functions/manage-functions#set_timeout_and_memory_allocation.
18:35:30 W
function[us-central1-upperCaseMe]
Your function was killed because it raised an unhandled error.
```

`I` is "information, `W` means "warning".

For reasons I don't understand, a callable function executes twice.

You can see the log from the line `functions.logger.log('upperCaseMe', original, uppercase);`. You could use `console.log` instead. The [Cloud Functions logger SDK](https://firebase.google.com/docs/functions/writing-and-viewing-logs) has many features specific to Firebase. 

Finally the emulator throws an error: `Your function timed out after ~60s.` This seems to be a bug in the emulator. I ignore it.

### Data and context

`onCall` takes two parameters

*index.ts*
```js
functions.https.onCall((data, context)
```

`data` is the request we sent to the Cloud Function. `data.text` is our message.

`context` is information such as the user's userID, etc. More on this in the official documentation.

### `httpsCallable` vs. `httpsCallableFromURL`

Comment out the line with `httpsCallable` and comment in the line with `httpsCallableFromURL`. Change `my-projectId` in the URL to your `projectId` in `environment.ts`.

Run your code again. A Firebase team member told me, "`httpsCallableFromURL` is for cases when you cannot use the normal way that the SDK locates the backend, for example, if you are hosting the callable function on some service that is not a normal Cloud Functions backend, or you are trying to invoke a function in a project that's not part of your app."

In other words, you shouldn't need to use `httpsCallableFromURL`. However, I've twice had `httpsCallable` fail and `httpsCallableFromURL` work. In the first case, `httpsCallable` repeatedly threw a CORS error but `httpsCallableFromURL` executed without error. Then when I tried `httpsCallable` again it executed without the CORS error. 

In the second case, I'd initialized Firebase incorrectly (initialize Firebase in `app.module.ts`, not in `app.component.ts`!). `httpsCallableFromURL` executed correctly. The emulator logs showed that `httpsCallable` executed correctly but in the browser console the Cloud Function returned `null`.

I also like that `httpsCallableFromURL` allows switching between the emulator and the cloud by changing the URL. But again, there's a better way to do this (in `environment.ts` change `production` between `true` and `false`).

My conclusion is that if your callable function isn't working, try `httpsCallableFromURL` and your function might work. But don't stop there. Figure out what the problem was with `httpsCallable` and fix it.

### CORS errors

As noted above, sometimes you get a CORS error when calling a Cloud Function from Angular running locally:

```
Access to fetch at 'https://us-central1-my-project.cloudfunctions.net/myFunction' from origin 'http://localhost:4200' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

This (StackOverflow question)[https://stackoverflow.com/questions/42755131/enabling-cors-in-cloud-functions-for-firebase] has lots of solutions to this problem. 

One solution that worked for me was to make the Cloud Function publically callable. This is a security risk so should only be done for development. Go to your (Google Cloud Console Dashboard)[https://console.cloud.google.com/home/dashboard]. Go to Resources, then Cloud Functions. From your list of Cloud Functions, click on the function, then the PERMISSIONS tab, click GRANT ACCESS, then under Add principals type in `allUsers`, then under Assign roles select Cloud Functions and then Cloud Function Invoker and then SAVE.

#### The emulator URL

With `httpsCallableFromURL` the URL is crucial. With the emulator, the URL has four parts:

* The URL of the server. The port (`5001`) was provided when you started the emulator:

```
┌───────────┬────────────────┬─────────────────────────────────┐
│ Emulator  │ Host:Port      │ View in Emulator UI             │
├───────────┼────────────────┼─────────────────────────────────┤
│ Functions │ 127.0.0.1:5001 │ http://127.0.0.1:4000/functions │
```

This URL also works:

```
httpsCallableFromURL(this.functions, 'http://127.0.0.1:5001/languagetwo-cd94d/us-central1/upperCaseMe');
```

* The `projectId`. This is in `environments/environment.ts` or in Project Settings in your Firebase console. You can also get it from the CLI with `firebase use`.
* The server's `locationId` plus `1`. This is also in `environments/environment.ts`.
* The name of the Cloud Function.

The emulator doesn't provide the URL. (It should, IMHO.)

#### The Firebase Cloud URL

This URL is available in your Firebase Console in your list of Functions. It should look similar to:

```js
httpsCallableFromURL(this.functions, 'https://us-central1-myprojectId.cloudfunctions.net/upperCaseMe');
```

## Trigger your Cloud Function

The third way to execute a Cloud Function is to write to Firestore or another Firebase database.

Copy and paste this Cloud Function into your `index.ts`:

*index.ts*
```js
export const triggerMe = functions.firestore.document('Messages/{docId}').onCreate((snap, context) => {
    const original: string = snap.data().original;
    const uppercase: string = original.toUpperCase();
    functions.logger.log('Uppercasing', original, uppercase);
    return snap.ref.set({ uppercase }, { merge: true });
});
```

Go to your emulator [http://127.0.0.1:4000/functions](http://127.0.0.1:4000/functions) and click on the Firestore tab and `+ Start collection`. Call the collection `Messages`. A document ID is automatically generated. In the document field enter `original` as the key and a `uppercase` as the value. Wait a moment and you should see a new field appear: `uppercase: UPPERCASE`.

Let's look at the difference between the callable function and the triggerable function. With the triggerable function you must specify a document location in Firestore. Here we're specifying the collection `Messages` and then using a wildcard for the documentId. 

Next, we use `onCreate`, which is for new documents. There are four  keywords we could use: 

* `onCreate`	Triggered when a document is written to for the first time.
* `onUpdate`	Triggered when a document already exists and has any value changed.
* `onDelete`	Triggered when a document with data is deleted.
* `onWrite` Triggered when `onCreate`, `onUpdate` or `onDelete` is triggered.

Next, the parameters are `snap` and `context`, not `data` and `context`. In the body of the Cloud Function we change `data.text` to `snap.data().original`.

The last difference is the `return`. Instead of returning the UPPERCASE message to the sender, the UPPERCASE message is written to Firestore. In this case the data is written to the document that triggered the function.

### Path to your trigger

This is not the path to your Firestore trigger:

```js
export const TriggerMyCloudFunction = functions.firestore.document('Thing').collection('Things').document('{thingID}').onCreate((snap, context) => {
```

This is the path to your Firestore trigger:

```js
export const TriggerMyCloudFunction = functions.firestore.document('Thing/Things/{thingID}').onCreate((snap, context) => {
```

The path to a Firestore location doesn't look like the path to a Firestore location in the front end.

There's lots more about triggerable Cloud Functions in the [official documentation](https://firebase.google.com/docs/functions/firestore-events), plus triggers from Auth, Cloud Storage, and other Firebase databases.

# Writing Cloud Functions

Now you know how to setup Cloud Functions and then call or trigger a Cloud Function. It's time to write some Cloud Functions.

## Node.js

Cloud Functions for Firebase is a [Node.js](https://nodejs.org/en/) runtime. You can run any library in Cloud Functions and write your code any way you want. Typically we use the Firebase Admin SDK or the Google Cloud SDK. To rephrase that, there isn't a list of "Cloud Functions commands" that you have to use.

### Check your Node version

Open `functions/package.json` and check the version of Node:

*functions/package.json*
```js
  "engines": {
    "node": "16"
  },
```

Cloud Functions currently use Node 16. This will change at some point to Node 18.

### `require` vs. `import`

The TypeScript transpiler will reduce ES module syntax (`import`) to CommonJS module syntax (`require`). It's OK to mix these in your `index.ts`. 

I recommend *not* adding `"type": "module"` to your `package.json`. This will use ES modules in your Node.js runtime, i.e., your `index.js` will be an ES module. If you're using TypeScript you want `index.ts` to be an ES module and your `index.js` to be a CommonJS module.

If you're writing your Cloud Functions in JavaScript use the CommonJS module syntax (`require`).

## Initialize `admin` with Firebase Admin SDK v9 nested namespace hierarchy syntax

*index.ts*
```
import * as functions from "firebase-functions";
const admin = require('firebase-admin');
admin.initializeApp();
```

This enables you to use syntax starting with `admin`, a.k.a, the nested namespace hierarchy with the global `admin()` at the top of the hierarchy.

### Initialize Firebase Admin SDK v10 modular syntax (DON'T USE)
On October 14, 2021, the Firebase team released [Firebase Admin Node.js SDK v10](https://firebase.google.com/docs/admin/migrate-node-v10). The new syntax imports only the modules your Cloud Function needs, making your app download faster. This is a good idea for your front end app but has no advantage for your back end, which is never downloaded to a client computer or mobile device.

Initialize your `index.ts`.

*index.ts*
```js
import { initializeApp, App } from 'firebase-admin/app';
import { getFirestore } from 'firebase-admin/firestore'
import { getFunctions, Functions } from 'firebase-admin/functions';
const app: App = initializeApp();
const firestore = getFirestore(app);
const functions: Functions = getFunctions(app);
```

As far as I can tell that's as far as you can get in Cloud Functions with Firebase Admin SDK v10. You can't call a Cloud Function with `onCall` or trigger a Cloud Function with `onCreate` using Firebase Admin SDK v10 modular syntax.

Just to confuse you, the Firestore documentation says that Web version 9 is modular and Web version 8 is namespaced, when in the Firebase Admin SDK version 10 is modular and version 9 is namespaced.

## Asynchronous operations with `async await` or promises

Database operations are asynchronous. Structure your database calls with either `async await` or promises:

*index.ts*
```js
export const upperCaseMe = functions.https.onCall(async (data: any, context: any) => {
    const original: string = data.text;
    const uppercase: string = original.toUpperCase();
    functions.logger.log('upperCaseMe', original, uppercase);
    await admin.firestore().collection('Messages').add({ original, uppercase });
    return uppercase;
});
```

Note that `async` goes into the parameters of `onCall`.

Promises:

*index.ts*
```js
admin.firestore().collection('MyCollection').doc('MyDocument').get()
  .then(function(doc) {
        if (doc.exists) {
          console.log("Document found.");
        } else {
          console.log("No such document.");
        }
   .catch (error) {
        console.error(error);
   }       
```

## Terminate Cloud Functions with `return`

All Cloud Functions must terminate with `return`. Even if you don't need anything returned, terminate with `return 0`. If you don't do this you'll see an error in your logs.

[`return` is synchronous](https://firebase.google.com/docs/functions/terminate-functions). Don't expect a callable function to return the results of an asynchronous operation. Use a promise to return asynchronous results after `return` is returned. If you try to return the results of an asynchronous call you are likely to see `null`.

## Cloud Firestore `get()`, `set()`, `update()`, `delete()`

*Do not* use the Firebase Web version 9 methods `setDoc`, `addDoc`, `updateDoc`, or `deleteDoc`. These methods will cause the TypeScript transpiler to make a [mess of your directory structure](https://stackoverflow.com/questions/74831161/setdoc-write-to-firestore-from-typescript-functions-messes-up-directory-struct), adding new directories and files that shouldn't be there.

Instead, use these [Node.js Server SDK for Google Cloud Firestore](https://cloud.google.com/nodejs/docs/reference/firestore/latest) methods to write Cloud Functions with the Firestore database:

* set()
* update()
* get()
* delete()

### CREATE: `set`

Mostly I use Cloud Functions to get data from APIs then write the data to Firestore or Storage. 90% of my Cloud Functions use `set` to write data to these databases. This code writes an object to a document. 

*index.ts*
```js
admin.firestore().collection('Users').doc(context.params.userID).collection(longLanguage).doc('Word_Response_Forvo').set({
        'word': wordObject})
        .then() // do nothing
        .catch(error => console.error(error));
})
```

Here's a similar operation.

*index.ts*
```js
async function buildWordObject() {
    try {
        const wordObject = {
            addedByName: 'DictionaryBuilder',
            dateAdded: yearMonthDay,
            frequency: wordFrequency,
            longLanguage: longLanguage,
            requestOrigin: requestOrigin,
            shortLanguage: shortLanguage,
            timeAdded: Date.now(),
            word: word,
          };
          await admin.firestore().collection('Dictionaries').doc(longLanguage).collection('Words').doc(word).set(wordObject);
    } catch (error) {
          console.error(error);
    }
}
buildWordObject();
```

Here's a `set` operation that uses `{ merge: true }` to create a new document if none exists or update an existing document.

*index.ts*
```js
admin.firestore().collection('Dictionaries').doc('Spanish').collection('Words').doc(change.after.data().word).collection('Translations').doc('English').set({
        translationsArray: translationsArray,
        shortlanguage: 'en',
        longLanguage: 'English',
        source: source,
        timeAdded: Date.now(),
        dateAdded: yearMonthDay
}, { merge: true });
```

### UPDATE: `update`

`update` will update an existing document. If the document doesn't exist it won't do anything.

```js
admin.firestore().collection('Dictionaries').doc('Spanish').collection('Words').doc(word).collection('Pronunciations').doc(pronunciation).update( {audioFile: url} )
    .then(function() {
        console.log("Download URL updated to database.");
    })
    .catch(function(error) {
        console.error(error);
    });
```

### READ: `get`

To read data from Firestore use `get`. This code gets a document from Firestore.

*index.ts*
```
admin.firestore().collection('Dictionaries').doc('Spanish').collection('Words').doc(word).collection('Pronunciations').doc(pronunciation).get()
  .then(function(doc) {
            if (doc.exists) {
              console.log("Document found.");
            } else {
              console.log("No such document.");
            }
   .catch (error) {
        console.error(error);
   }       
```

This code uses a filter to check if the current user is listed in the collection `Trusted_Users`.

*index.ts*
```
admin.firestore().collection('Trusted_Users').where('UID', '==', userID).get()
  .then(function(querySnapshot) {
    querySnapshot.forEach(function(doc) {
      console.log("Trusted user: ", doc.id, " => ", doc.data());
    });
     })
  .catch(function(error) {
    console.log("Error getting documents: ", error);
  });
```

### DELETE: `delete`

This will delete a document.

*index.ts*
```js
admin.firestore().collection('MyCollection').doc('MyDocument').delete()
```

### Other methods

There may be other methods, such as `add`. The difference between `add` and `set` is that `add` makes a new document when `set` writes data to an existing document that you've identified by its `documentID`.

Another method I've used is `listDocuments()`. This code lists the documents in a collection.

*index.ts*
```js
admin.firestore().collection('Videos').doc(longLanguage).collection('Translations').listDocuments()
        .then(function(value) {
          value.map((value) => {
            value.delete();
            resolve('Deleted.');
          });
        })
        .catch(function(error) {
          console.error(error);
          reject(error);
        });
```

## Cloud Storage

Cloud Firestore stores data: objects, arrays, strings, numbers, etc. Cloud Storage stores files. Handling files is very different from handling data.

There's no set of commands like `set()`, `get()`, `update()`, `delete()`. Instead, you use Node.js to handle files. Node.js has a steep learning curve. The Firebase team put together [code samples](https://cloud.google.com/nodejs/docs/reference/storage/latest) for just about anything you might want to do with Cloud Storage. 

### `uploadBytes` from your app or front end

Cloud Storage has [easy to use commands](https://firebase.google.com/docs/storage/web/upload-files):` uploadBytes()` and `uploadString()`. `uploadBytes` is a method for writing a file to Storage. These commands are for your app or front end. Consider structuring your code to handle files from your app or front end. For example, your app calls a Cloud Functions that calls an API to get a file, and then the Cloud Functions writes the file to Cloud Storage. If you can instead get a download URL then pass the download URL from your Cloud Function back to your app or front end, and then use `uploadBytes()` to write the file to Cloud Storage.

### AngularFire Storage methods for your app or front end

AngularFire Storage has lots of methods.

```js
import { connectStorageEmulator, deleteObject, fromTask, getBlob, getBytes, getDownloadURL, getMetadata, getStorage, getStream, list, listAll, percentage, provideStorage, ref, storageInstance$, updateMetadata, uploadBytes, uploadBytesResumable, uploadString } from '@angular/fire/storage';
```

These methods, which are in AngularFire 7.5, aren't in the [AngularFire Storage documention](https://github.com/angular/angularfire/blob/master/docs/storage/storage.md), which appears to be written for AngularFire 6.

# Deploy your Cloud Function to Firebase

To call the Cloud Function in the Firebase Cloud, first deploy your Cloud Function from your `functions` folder:

```
firebase deploy --only functions:upperCaseMe
```

That deploys just one function. To deploy all your functions:

```
firebase deploy --only functions:
```

To deploy your entire project:

```
firebase deploy
```

Wait a few minutes and you should see your Cloud Function listed in your Firebase Console. If you run it you'll see the logs in your Firebase Console.

## Manage your credentials

Putting API keys and other credentials in your `index.ts` may not be a good idea. You might push your code to public repository on GitHub, exposing your credentials. It's better to put your credentials into `environment.ts`.

Cloud Functions have a way to manage credentials with [parameterized configuration](https://firebase.google.com/docs/functions/config-env#params). I haven't tried this, I will work on this.

### Use an IAM service account

Go to `https://console.cloud.google.com/iam-admin/iam`. 

Look for a service account named "App Engine default service account".

Use only this service account with your Cloud Functions. Add new roles to this service account as needed. Don't make new service accounts.

To add a new role, click the pencil icon to the right of the service account. Then click "Add another role". Then save the changes.

### Hook up your service account to your Cloud Functions

Hook up a service account by downloading the key and then initializing `admin` with the credentials:

*index.ts*
```js
admin.initializeApp({
  apiKey: 'abc123', 
  authDomain: 'my-project.firebaseapp.com', 
  credential: admin.credential.cert({
    "type": "service_account",
    "project_id": "my-project",
    "private_key_id": "123abc",
    "private_key": "-----BEGIN PRIVATE KEY-----\123abc\-----END PRIVATE KEY-----\n",
    "client_email": "firebase-adminsdk-1234x@my-project.iam.gserviceaccount.com",
    "client_id": "0987654321",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/firebase-adminsdk-1234x%40my-project.iam.gserviceaccount.com"
  }),
  databaseURL: 'https://my-project.firebaseio.com',
  storageBucket: 'gs://my-project.appspot.com', 
});
```

The security of this is questionable. Instead, try to put your service key into your `environments` folder, perhaps in a `service-accounts` folder. Then import the service key:

*index.ts*
```js
const serviceAccount = require('../../environments/service_account_keys/my-project-firebase-adminsdk-1234x-abcd.json');

admin.initializeApp({
  apiKey: 'abc123', 
  authDomain: 'my-project.firebaseapp.com', 
  credential: admin.credential.cert(serviceAccount),
  databaseURL: 'https://my-project.firebaseio.com',
  storageBucket: 'gs://my-project.appspot.com', 
});
```

You must add `"resolveJsonModule": true,` to the `compilerOptions` in your `tsconfig.json`.

I can't get this to work. The only way that works for me is to out the service account key into `index.ts`.

### Deploy a service account with `gcloud`

This is a questionable practice.

If you're using JavaScript Cloud Functions you can connect a service account to a Cloud Function by deploying the function as a Google Cloud Function:

```
gcloud functions deploy upperCaseMe --service-account google-cloud-translate@my-projectId.iam.gserviceaccount.com
```

If you do this, no credentials are needed in your app. The service account works like magic!

Firebase deploy won't hook up a service account. This doesn't work:

```
firebase deploy --only functions:upperCaseMe --service-account google-cloud-translate@my-projectId.iam.gserviceaccount.com
```

`gcloud functions deploy` doesn't work with TypeScript. It looks for `lib/index.js` in the `src` directory, not in the parent directory.

### The "Project 563584335869" error

I was getting this error with Cloud Functions in the Firebase Emulators:

```
Cloud Translation API has not been used in project 563584335869 before or it is disabled. Enable it by visiting https://console.developers.google.com/apis/api/translate.googleapis.com/overview?project=563584335869 then retry.
```

Going to that URL throws an error: `You do not have sufficient permissions to view this page`. 

I'll nominate that error message for Misleading Error Message of the Month. It's saying that your service account is missing a role. Go to your Google Cloud Console and check if your Cloud Functions service account has all necessary roles, including `Firebase Admin`.

# Use Functions in your Firebase Console

You can view you functions, read the logs, get each function's URL, and delete functions from your Firebase Console.

# Testing Cloud Functions

I need to learn this.
