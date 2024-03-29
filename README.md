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

#### Where's my server location?

Occasionally you'll need to write a URL which includes your server location. This is not in your Firebase console settings. Your server location can be seen in your list of functions in the Firebase console.

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

You can trigger a Cloud Function by writing to your database from your app or front-end framework but this isn't as good. It's more code, especially if you want to return data to your app or front end. And writing to your database may be slower. If your app or front-end wants to get data from an API, when the data comes back your Cloud Function has to write the data to your database. Then you set up an Observer in your app or front-end to listen for changes in your database. This is a lot of code. Then when the data comes back you'll want to unsubscribe the listener. I haven't found a clean way to unsubscribe the listener when the data comes back.

If you want to use a listener for a long time, i.e., for many data changes in your database, then triggering a Cloud Function with a database write might make sense. But if your app or front-end calls a Cloud Function to get data once from an API, this isn't what listeners are good at. It's better to use a HTTPS callable Cloud Function.

If your Cloud Function triggers from events in your database, not from your app or front-end, this is where you use a triggerable Cloud Function.

## HTTPS callable functions

Copy and paste this Cloud Function in `index.ts`:

*index.ts*
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
    connectFunctionsEmulator(this.functions, "127.0.0.1", 5001);
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

A callable function executes twice. I don't know why.

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

One difference between `httpsCallable` and `httpsCallableFromURL` is that `httpsCallable` will call the cloud function in the Google cloud unless you specify, a the top of the cloud function, that you want to call the emulator with `connectFunctionsEmulator()`:

*app.component.ts*
```js
 callMe() {
    connectFunctionsEmulator(this.functions, "127.0.0.1", 5001); // <-- this line
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
```

Comment out `connectFunctionsEmulator()` when you want to run your function in the cloud. 

A Firebase team member told me, "`httpsCallableFromURL` is for cases when you cannot use the normal way that the SDK locates the backend, for example, if you are hosting the callable function on some service that is not a normal Cloud Functions backend, or you are trying to invoke a function in a project that's not part of your app."

In other words, you shouldn't need to use `httpsCallableFromURL`. However, I've twice had `httpsCallable` fail and `httpsCallableFromURL` work. In the first case, `httpsCallable` repeatedly threw a CORS error but `httpsCallableFromURL` executed without error. Then when I tried `httpsCallable` again it executed without the CORS error.

In the second case, I'd initialized Firebase incorrectly (initialize Firebase in `app.module.ts`, not in `app.component.ts`!). `httpsCallableFromURL` executed correctly. The emulator logs showed that `httpsCallable` executed correctly but in the browser console the Cloud Function returned `null`.

I also like that `httpsCallableFromURL` allows switching between the emulator and the cloud by changing the URL. But again, there's a better way to do this (in `environment.ts` change `production` between `true` and `false`).

My conclusion is that if your callable function isn't working, try `httpsCallableFromURL` and your function might work. But don't stop there. Figure out what the problem was with `httpsCallable` and fix it.

### CORS errors

As noted above, sometimes you get a CORS error when calling a Cloud Function from Angular running locally:

```
Access to fetch at 'https://us-central1-my-project.cloudfunctions.net/myFunction' from origin 'http://localhost:4200' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

First, this error message often is thrown by something other than a CORS error. In other words, you could go down a rabbit hole looking for a CORS error. If you call a function in the cloud before you've deployed the function, or in the emulator before you've compiled the function (`npm run build`), you'll get a CORS error. If you call a function in the emulator without connecting the emulator (`connectFunctionsEmulator()`) you'll get a CORS error.

There are multiple ways to fix a CORS error, depending on your situation.

#### `{ cors: true }` in `onRequest` 2nd generation functions

Second generation `onRequest` functions can handle CORS:

```js
export const UppercaseMe2ndGen = onRequest({ cors: true }, (request) => {
```

You can set up rules such as allowing only requests from your Flutter app.

#### IAM service accounts

https://stackoverflow.com/questions/73821144/authorize-access-to-google-cloud-translate-from-firebase-cloud-function

I don't understand how IAM service accounts fix CORS problems. They might not, perhaps I'm ascribing magical powers to service accounts!

If your Cloud Functions use a service account for your credentials you must provide principals and roles to invoke your Cloud Functions.

Go to your [Google Cloud Console Dashboard](https://console.cloud.google.com/home/dashboard). Go to *Resources*, then *Cloud Functions*. From your list of Cloud Functions, click on the function, then the *PERMISSIONS* tab, click *+GRANT ACCESS*, then under *Add principals* type in `allUsers`, then under *Assign roles* select *Cloud Functions* and then *Cloud Function Invoker* and finally *SAVE*.

This makes your Cloud Function publicly accessible, which is a security risk. These articles explain how to write code in the body of your function to check whatever form of authentication is being provided by the caller, which for callables, is going to be Firebase Auth.

[How to restrict Firebase Cloud Function to accept requests only from Firebase Hosting website](https://stackoverflow.com/questions/69291334/how-to-restrict-firebase-cloud-function-to-accept-requests-only-from-firebase-ho)

[Firebase functions authorize only requests from Firebase hosting app](https://stackoverflow.com/questions/67649115/firebase-functions-authorize-only-requests-from-firebase-hosting-app)

#### Set "Access-Control-Allow-Origin" in firebase.json

The CORS error message asks you to put `Access-Control-Allow-Origin` in the headers. If you host your Angular app on Firebase you can do this in `firebase.json`

*firebase.json*
```js
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "headers": [ {
      "source": "/myCloudFunction",
      "headers": [ {
        "key": "Access-Control-Allow-Origin",
        "value": "*"
      } ]
    } ]
  },
```

This didn't work for me when my Angular app was running locally at `http://localhost:4200/home`.

#### Import `cors` with `onRequest`

If you call your Cloud Function with `onRequest` (not `onCall` or `onCreate`) you can import the npm module `cors` into your Cloud Functions. The following code is for JavaScript. [This question](https://stackoverflow.com/questions/42755131/enabling-cors-in-cloud-functions-for-firebase) shows how to use this in TypeScript.

*index.js*
```
const cors = require('cors')({origin: true});

exports.fn = functions.https.onRequest((req, res) => {
    cors(req, res, () => {
        // your function body here - use the provided req and res from cors
    })
});
```

#### Call GoogleAuth, which calls your Cloud Function

Here's another way to fix a CORS error. This looks complicated but works well when your Cloud Function accesses a Google Cloud API (e.g., Cloud Translate). 

First, your Angular front-end app calls the GoogleAuth app. A request object is sent with several properties. We also make a Firestore listener to get the results back.

```js
async callMe() {
  const Call_Google_Romanize = httpsCallableFromURL(this.functions, 'https://us-central1-languagetwo-cd94d.cloudfunctions.net/Call_Google_Romanize_HTTP');
  // const Call_Google_Auth = httpsCallableFromURL(this.functions, 'https://us-central1-languagetwo-cd94d.cloudfunctions.net/Call_Google_Auth');

    const request = {
      contents: ["لا تقلل من شأن أي شخص"], // must be an array
      source_language_code: "ar",
      longLanguage: "Arabic",
    };

    let response = await Call_Google_Auth(request);

    // make listener
    const unsub = onSnapshot(doc(this.firestore, 'Dictionaries/' + this.language2.long + '/Words/' + request.contents), (doc) => {
      console.log("Current data: ", doc.data());
    });
}
```

In `index.js` let's make the second Cloud Function first.

*index.js*
```js
export const Call_Google_Romanize_HTTP = onRequest(async (request: any, response: any) => {
  const options = {
    headers: {
      "Authorization": "Bearer 12345abcde",
      // "Authorization": "Bearer " + request.body.token,
      "x-goog-user-project": "languagetwo-cd94d",
      "Content-Type": "application/json; charset=utf-8",
    },
    json: {
      "source_language_code": request.body.source_language_code,
      "contents": request.body.contents,
    }
  };

  response = await got.post("https://translation.googleapis.com/v3/projects/languagetwo-cd94d/locations/us-central1:romanizeText", options).json()
    .then((response: any) => {
      admin.firestore().collection('Dictionaries').doc(request.body.longLanguage).collection('Words').doc(request.body.contents[0]).set({
        'romanizedText': response.romanizations[0].romanizedText
      })
        .then() // do nothing
        .catch(error => logger.error(error));
    })
    .catch((error: any) => {
      logger.log(error);
    });
});
```

The first code block sets up the API call. The key line is `"Authorization":`. This is the *access code* for the API. Generate a manual access code from the CLI:

```
gcloud auth application-default print-access-token
```

This [documentation](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/print-access-token) explains manually generating access codes.

Cut and paste your access code after `"Authorization":` preceded by `Bearer`. Bearer means this is a bearer token.

Call your Cloud Function from your front end Angular app and check that it works. You should see the results in your front end console.

Your access code token will last one hour. Let's programmatically generate a new access code every time we call the Cloud Function.

Make another Cloud Function before the previous function.

```js
export const Call_Google_Auth = onCall(async (request) => {
  const targetAudience = 'https://us-central1-languagetwo-cd94d.cloudfunctions.net/Call_Google_Romanize_HTTP';
  const url = targetAudience;

  const { GoogleAuth } = require('google-auth-library');
  const auth = new GoogleAuth();

  const token = await new GoogleAuth({
    scopes: ["https://www.googleapis.com/auth/cloud-platform"],
  }).getAccessToken();
  request.data.token = token;

  async function authRequest() {
    const client = await auth.getIdTokenClient(targetAudience);
    const res = await client.request({
      url: url,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization',
      },
      body: JSON.stringify(request.data),
    })
    console.info(res.body);
  };

  authRequest()
    .then(() => {
      return 0;
    })
    .catch(err => {
      console.error(err.message);
      process.exitCode = 1;
      return 0;
    });
});
```

This [documentation](https://cloud.google.com/functions/docs/securing/authenticating#generating_tokens_programmatically) explains programmatically generating bearer tokens.

In your front end Angular app, comment out the call to the second Cloud Function and comment in your GoogleAuth Cloud Function.

`targetAudience` is the URL of your second Cloud Function. 

We call `GoogleAuth()` twice. First, we call with `getAccessToken()`, then put the result onto the request data object.

Next, we call with `getIdTokenClient()`. We then use this ID token we can call the second Cloud Function.

Why two types of tokens? There are five or six [token types](https://cloud.google.com/docs/authentication/token-types):

* Access tokens
* ID tokens
* Self-signed JWTs
* Refresh tokens
* Federated tokens
* Bearer tokens

ID tokens are special type of JavaScript Web Token (JWT). Access tokens, ID tokens, and JWTs are all bearer tokens. Here we need an *ID token* for the first Cloud Function to call the second Cloud Function. We need an *access token* for the second Cloud Function to call Cloud Translate. The access token is passed from the first Cloud Function to the second Cloud Function.

This prevents a CORS error but it's now impossible to return the results from then second Cloud Function to the front end Angular app. We have to write the results to Firestore and make a listener in the front end Angular app.

## The emulator URL

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

#### Writing to the emulator vs. writing to the cloud

If your Cloud Function is called with the emulator URL, it will write to Firestore (or Storage) in the emulator.

If your Cloud Function is called with the Firebase cloud URL, it will write to Firestore (or Storage) in the cloud.

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

## Terminate Cloud Functions and return results from callable Cloud Functions: Synchronous vs async

Cloud Functions must be [terminated](https://firebase.google.com/docs/functions/terminate-functions) to avoid excessive billing charges. 

Synchronous callable Cloud Functions can easily return results to the front end function that called them. Async callable Cloud Functions can't return results to the front end, unless you set up an Observer for Firestore. 

Triggerable async Cloud Functions can't return anything to the front end, unless you set up an Observer for Firestore. 

Cloud Function operations with Storage can't be observed or returned to the front end.

### In Angular

*component.ts*
```js
callMe() {
    const callIBMIPA = httpsCallable(this.functions, 'Call_IBM_IPA');
    callIBMIPA({
      format: 'ipa',
      voice: voices[0].voice,
      word: word,
    })
     .then((pronunciation) => {
        console.log(pronunciation.data as string)
    });
}
```

The Angular component sets up a call to a cloud function, calls the cloud function, then uses a promise to get the result. No `async await` is needed in the Angular handler function.

### Return synchronous results and terminate Cloud Function

All Cloud Functions must terminate with `return`. Even if you don't need anything returned, terminate with `return 0`. If you don't do this you'll see an error in your logs.

Use `return` to send synchronous results to the front end function that called the Cloud Function. The `upperCaseMe` Cloud Function does this.

*index.ts*
```js
export const upperCaseMe = functions.https.onCall((data, context) => {
  try {
    const original = data.message;
    const uppercase = original.toUpperCase();
    functions.logger.log('upperCaseMe', original, uppercase);
    return uppercase;
  } catch (error) {
    console.error(error);
    return 0;
  }
});
```

This Cloud Function returns an UPPERCASE string, if successful, or returns 0 if unsuccessful.

### Asynchronous Cloud Functions

Returning a result from an async cloud function is challenging. `return` is synchronous and will usually execute before the async results come back. But we can use `async await` to return async results. Let's examine an example.

*index.ts*
```js
export const Call_IBM_IPA = onCall(async (request) => {
   ...
   let results: string | null = null;
   await textToSpeech.getPronunciation(getPronunciationParams)
     .then(pronunciation => {
       results = pronunciation.result.pronunciation;
     })
     .catch(error => {
        return 'error:' + error
     });
   return results;
}
```

First, note where `async` goes. We'll skip the code that sets up the API call. We then make a variable to hold the results of the API call. Then we have `await` and the API call, which returns a promise. When the results come in we assign them in the `results` variable. `return` then sends the results to the Angular front end handler function that called the cloud function.

Let's look at another example. This returns `null` to the Angular handler function, writes the results to the database, and logs "Write succeeded!" to the Cloud Functions console. It returns `null` to the front end because we're not using `async await`.

*index.ts*
```js
export const writeUppercase2FirestorePromise = functions.https.onCall((data: any, context: any) => {
  const original: string = data.message;
  const uppercase: string = original.toUpperCase();
  return admin.firestore().collection('Messages').add({ original, uppercase })
    .then(() => {
      logger.log('Write succeeded!');
    })
    .catch((error: any) => {
      logger.error(error);
    });
});
```

Another example of `async await`.

*index.ts*
```js
export const writeUppercase2FirestoreAsyncAwait = functions.https.onCall(async (data: any, context: any) => {
  try {
    const original: string = data.message;
    const uppercase: string = original.toUpperCase();
    await admin.firestore().collection('Messages').add({ original, uppercase });
    logger.log(uppercase);
    return uppercase;
  } catch (error) {
    logger.error(error);
    return error
  }
});
```

And another more example of how not to return async results to the front end.

*index.ts*
```js
export const getUppercase2FirestoreAsyncAwait = functions.https.onCall((data: any, context: any) => {
  admin.firestore().collection('Messages').doc('Ry7mrsEn7F1mNC8obM3H').get()
    .then((doc) => {  // get the document with the ID "Ry7mrsEn7F1mNC8obM3H"));
      if (doc.exists) {
        console.log("Document data:", doc.data());
        return doc.data();
      } else {
        // doc.data() will be undefined in this case
        console.log("No such document!");
        return null;
      }
    }).catch((error) => {  // catch any errors
      console.log("Error getting document:", error);
      return null;
    });
});
```

This will return `null` to the front end because we're not using `async await`.

How to return results from Firestore in a nested function.

*index.ts*
```js
export const Call_Word_Request = onCall(async (request: any) => { // <-- async callable function
  const language1: Language = request.data.language1;
  const language2: Language = request.data.language2;
  const word: string = request.data.word;
  const requestOrigin: string = request.data.requestOrigin;
  let thousandsOfWords: string[] = thousandsOfEnglishWords
  let results: any = null; // <-- results to be returned to Angular

  async function writeWordObjectToFirestore() { // <-- async inner function
    await admin.firestore().collection('Dictionaries').doc(language2.long).collection('Words').doc(word).set({ // <-- call Firestore and await for result
      addedByName: 'DictionaryBuilder',
      dateAdded: yearMonthDay,
      frequency: wordFrequency,
      longLanguage: language2.long,
      requestOrigin: requestOrigin,
      shortLanguage: language2.short,
      timeAdded: Date.now(),
      word: word,
    }, { merge: true })
      .then(result => { // <-- result from Firestore
        results = result.writeTime; // {_seconds: 1685047282, _nanoseconds: 305252000}
      })
      .catch(error => {
        logger.error(error);
      });
      return results // <-- return results from inner function to cloud function
  };

  return await writeWordObjectToFirestore(); // <-- await for results from inner function, return results to Angular
}); 
```

Finally, if all else fails, write the result to Firestore and make a listener in your front end Angular app. The section on CORS errors has an example of this.

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

Cloud Firestore stores data: objects, arrays, strings, numbers, etc. Cloud Storage stores large digital files, such as photos, music, and videos. Handling files is very different from handling data.

There's no set of commands like `set()`, `get()`, `update()`, `delete()`. Instead, you use Node.js to handle files. Node.js has a steep learning curve. The Firebase team put together [code samples](https://cloud.google.com/nodejs/docs/reference/storage/latest) for just about anything you might want to do with Cloud Storage. 

### Get a Storage bucket

*index.js*
```js
import { Storage } from '@google-cloud/storage'; // Imports the Google Cloud client library
const storage = new Storage(); // Creates a client
const bucketName = 'gs://languagetwo-cd94d.appspot.com'; // The ID of your GCS bucket
```

This code goes at the top of your `index.js` file, above the Cloud Functions. It sets up your Storage bucket.

#### Saving with `rawBody`

This function uses an undocumented feature `rawBody`.

```js
const destFileName = 'Audio/' + language2.long + '/' + word + '/' + pronunciation + '/' + wordFileType; // write to this location in Storage
async function uploadFromMemoryToStorage() {
      try {
        file = await got(pronunciationObject.oedAudioDownloadURL![0]); // download the audio file from OED; refactor for multiple pronunciations
        await storage.bucket(bucketName).file(destFileName).save(file['rawBody']); // write a file to Storage
        await storage.bucket(bucketName).file(destFileName).makePublic();  // make the audio file public. The first download will change this to a token download URL. See second answer at https://stackoverflow.com/questions/42956250/get-download-url-from-file-uploaded-with-cloud-functions-for-firebase
        let audioFile: string = 'https://storage.googleapis.com/languagetwo-cd94d.appspot.com/Audio/English/' + word + '/' + pronunciation + '/' + wordFileType;
        let audioFiles: string[] = [];
        audioFiles.push(audioFile);
        // admin.firestore.FieldValue.arrayUnion(audioFile) documentation: https://firebase.google.com/docs/reference/js/v8/firebase.firestore.FieldValue#arrayunion
        // FieldValues are sentinal values that indicate that something out of the ordinary has happened
        // arrayUnion returns a special value that can be used with set() or update() that tells the server to union the given elements with any array value that already exists on the server. Each specified element that doesn't already exist in the array will be added to the end. If the field being modified is not already an array it will be overwritten with an array containing exactly the specified elements.
        // await admin.firestore().collection('Dictionaries').doc(language2.long).collection('Words').doc(word).collection('Pronunciations').doc(pronunciation).set({ audioFiles: audioFiles }, { merge: true }); // this works adequately, the arrayUnion would be better but doesn,'t work, see if it works in the cloud
        // logger.log(admin.firestore.FieldValue.arrayUnion(audioFile));
        await admin.firestore().collection('Dictionaries').doc(language2.long).collection('Words').doc(word).collection('Pronunciations').doc(pronunciation).set({ audioFiles: admin.firestore.FieldValue.arrayUnion(audioFile) }, { merge: true }); // this version is intended to add new pronunciations to the array of audioFiles; FieldValue doesn't work in the emulator
        // error: admin.firestore.FieldValue is undefined

      } catch (error) {
        console.error(error);
      }
}

uploadFromMemoryToStorage()
```

First, a location is constructed in Storage to write to.

Then the function uses the npm library `got` to make an HTTP request and download the audio file.

Now we get to the fun part. We use `file(destFileName).save(file['rawBody'])` to write the file to Storage.

[rawBody](https://cloud.google.com/functions/docs/writing/write-http-functions#parsing_http_requests) appears to differentiate a response body from a response `rawBody` to handle a large digital file.

The rest is easy. The file is made public. Then the download URL is constructed and written to Firestore.

#### Firebase `getStorage()` method

There is a [firebase-admin.storage package](https://firebase.google.com/docs/reference/admin/node/firebase-admin.storage). The package has a method, `getStorage()`, for hooking up Storage in Cloud Functions.

*index.ts*
```js
admin.initializeApp({
  apiKey: 'abc123',
  ...
  storageBucket: 'gs://my-project.appspot.com',
});

const { getStorage } = require('firebase-admin/storage');
const bucket = getStorage().bucket();
```

This creates a `Storage` class with two methods, `Storage.app` and `Storage.bucket()`. The documentation doesn't give examples of using these methods. I don't know how to use these.

### Write to Storage with Node `file` 

```js
import textToSpeech = require('@google-cloud/text-to-speech');
export const Call_Google_Text_to_Speech = onRequest(async (request: any, response: any) => {
  try {
    const text = request.body.data.word;
    const longLanguage = request.body.data.language2.long;
    const languageCode = request.body.data.language2.short;
    const audioEncoding = request.body.data.audioEncoding;
    const voiceGender = 'NEUTRAL' // request.body.data.voiceGender; // doesn't work when data is passed through
    const voiceName = request.body.data.voiceName;

    const client = new textToSpeech.TextToSpeechClient();

    // construct the request to Google Cloud Text-to-Speech
    const voiceRequest = {
      input: { text: text },
      voice: { languageCode: languageCode, ssmlGender: voiceGender }, // name: 'en-US-Wavenet-D',
      audioConfig: { audioEncoding: audioEncoding },
    };

    const options = { // put in metadata
      metadata: {
        contentType: 'audio/mpeg',
        metadata: {
          source: 'Google Text-to-Speech'
        }
      }
    };

    const bucket = storage.bucket('languagetwo-cd94d.appspot.com');
    const destFileName = 'Audio/' + longLanguage + '/' + text + '/' + voiceName + '/' + text + '.mp3'; // write to this location in Storage
    var file = bucket.file(destFileName);

    let audioFile: string = 'https://storage.googleapis.com/languagetwo-cd94d.appspot.com/Audio/' + longLanguage + '/' + text + '/' + voiceName + '/' + text + '.mp3'; // download URL
    let audioFiles: string[] = [];
    audioFiles.push(audioFile);

    // Performs the text-to-speech request
    // @ts-ignore error TS2345: Argument of type 'voiceRequest' is not assignable to parameter of type 'ISynthesizeSpeechRequest'
    const [voiceResponse] = await client.synthesizeSpeech(voiceRequest); // await is necessary here despite what VSCode says
    return await file.save(voiceResponse.audioContent, options)
      .then(async () => {
        await storage.bucket(bucketName).file(destFileName).makePublic();
        console.log('Audio file written to Firebase Storage.');
        await admin.firestore().collection('Dictionaries').doc(longLanguage).collection('Words').doc(text).collection('Pronunciations').doc(voiceName).set({ audioFiles: admin.firestore.FieldValue.arrayUnion(audioFile) }, { merge: true }); // this version is intended to add new pronunciations to the array of audioFiles; FieldValue doesn't work in the emulator
      })
      .catch((error: any) => {
        logger.log(error);
      });
  } catch (error) {
    logger.log(error);
    return;
  }
});
```

This Cloud Function calls Google Cloud Text-to-Speech, downloads an audio file, and writes the audio file to Storage.

The first code block takes the request body passed from the front end and makes local variables for the text, language, etc. Then another code block constructs the request to send to Google Cloud Text-to-Speech.

The next code block adds metadata to the file.

The next code block sets up the Storage bucket and the location in Storage.

The next code block sets up the download URL. This code allows an array for multiple download URLs, for example, we could set up `.mp3` and `.webm` audio files.

Now we get to the fun part. This Cloud Function confuses VSCode. `// @ts-ignore` in the first line makes the Cloud Function run without TypeScript errors.

`client.synthesizeSpeech(voiceRequest)` calls Google Cloud Text-to-Speech and returns the audio file as `voiceResponse`.

`file.save(voiceResponse.audioContent, options)` is the crux of the Cloud Function. This creates a file to be stored, with the metadata. Somehow it saves the file to Storage. I don't see how.

After the promise is fulfilled, the `.then()` block makes the file public.

Finally, the download URL is written to Firestore. Note that `FieldValue.arrayUnion()`, which allows adding elements to an existing array, doesn't work in the emulator.

### `uploadBytes` from your app or front end

Cloud Storage has [easy to use commands](https://firebase.google.com/docs/storage/web/upload-files):` uploadBytes()` and `uploadString()`. `uploadBytes` is a method for writing a file to Storage. These commands are for your app or front end. Consider structuring your code to handle files from your app or front end. For example, your app calls a Cloud Function that calls an API to get a file, and then the Cloud Function writes the file to Cloud Storage. If you can instead get a download URL then pass the download URL from your Cloud Function back to your app or front end, and then use `uploadBytes()` to write the file to Cloud Storage.

### AngularFire Storage methods for your app or front end

AngularFire Storage has lots of methods.

```js
import { connectStorageEmulator, deleteObject, fromTask, getBlob, getBytes, getDownloadURL, getMetadata, getStorage, getStream, list, listAll, percentage, provideStorage, ref, storageInstance$, updateMetadata, uploadBytes, uploadBytesResumable, uploadString } from '@angular/fire/storage';
```

These methods, which are in AngularFire 7.5, aren't in the [AngularFire Storage documention](https://github.com/angular/angularfire/blob/master/docs/storage/storage.md), which appears to be written for AngularFire 6.

### Get a Storage download URL

Getting the download URL when uploading a file to Google/Firebase Cloud Storage is non-trivial. StackOverflow has many answers to [this question](https://stackoverflow.com/questions/42956250/get-download-url-from-file-uploaded-with-cloud-functions-for-firebase).

There are three types of download URLS:

 1. *signed* download URLs, which are temporary and have security features
 2. *token* download URLs, which are persistent and have security features
 3. *public* download URLs, which are persistent and lack security

There are three ways to get a token download URL. The other two download URLs have only one way to get them.

**From the Firebase Storage Console**

You can get the download URL from Firebase Storage console:

[![enter image description here][1]][1]

The download URL looks like this:

```
https://firebasestorage.googleapis.com/v0/b/languagetwo-cd94d.appspot.com/o/Audio%2FEnglish%2FUnited_States-OED-0%2Fabout.mp3?alt=media&token=489c48b3-23fb-4270-bd85-0a328d2808e5
```

The first part is a standard path to your file. At the end is the token. This download URL is permanent, i.e., it won't expire, although you can revoke it.

**getDownloadURL() From the Front End**

The [documentation][2] tells us to use `getDownloadURL()`:

```js
let url = await firebase.storage().ref('Audio/English/United_States-OED-' + i +'/' + $scope.word.word + ".mp3").getDownloadURL();
```

This gets the same download URL that you can get from your Firebase Storage console. This method is easy but requires that you know the path to your file, which in my app is about 300 lines of code, for a relatively simple database structure. If your database is complex this would be a nightmare. And you could upload files from the front end, but this would expose your credentials to anyone who downloads your app. So for most projects you'll want to upload your files from your Node back end or Google Cloud Functions, then get the download URL and save it to your database along with other data about your file.

**getSignedUrl() for Temporary Download URLs**

[getSignedUrl()][3] is easy to use from a Node back end or Google Cloud Functions:

```js
  function oedPromise() {
    return new Promise(function(resolve, reject) {
      http.get(oedAudioURL, function(response) {
        response.pipe(file.createWriteStream(options))
        .on('error', function(error) {
          console.error(error);
          reject(error);
        })
        .on('finish', function() {
          file.getSignedUrl(config, function(err, url) {
            if (err) {
              console.error(err);
              return;
            } else {
              resolve(url);
            }
          });
        });
      });
    });
  }
```
A signed download URL looks like this:

```
https://storage.googleapis.com/languagetwo-cd94d.appspot.com/Audio%2FSpanish%2FLatin_America-Sofia-Female-IBM%2Faqu%C3%AD.mp3?GoogleAccessId=languagetwo-cd94d%40appspot.gserviceaccount.com&Expires=4711305600&Signature=WUmABCZIlUp6eg7dKaBFycuO%2Baz5vOGTl29Je%2BNpselq8JSl7%2BIGG1LnCl0AlrHpxVZLxhk0iiqIejj4Qa6pSMx%2FhuBfZLT2Z%2FQhIzEAoyiZFn8xy%2FrhtymjDcpbDKGZYjmWNONFezMgYekNYHi05EPMoHtiUDsP47xHm3XwW9BcbuW6DaWh2UKrCxERy6cJTJ01H9NK1wCUZSMT0%2BUeNpwTvbRwc4aIqSD3UbXSMQlFMxxWbPvf%2B8Q0nEcaAB1qMKwNhw1ofAxSSaJvUdXeLFNVxsjm2V9HX4Y7OIuWwAxtGedLhgSleOP4ErByvGQCZsoO4nljjF97veil62ilaQ%3D%3D
```

The signed URL has an expiration date and long signature. The documentation for the command line [gsutil signurl -d][4] says that signed URLs are temporary: the default expiration is one hour and the maximum expiration is seven days. 

I'm going to rant here that [getSignedUrl][5] never says that your signed URL will expire in a week. The documentation code has `3-17-2025` as the expiration date, suggesting that you can set the expiration years in the future. My app worked perfectly, and then crashed a week later. The error message said that the signatures didn't match, not that the download URL had expired. I made various changes to my code, and everything worked...until it all crashed a week later. This went on for more than a month of frustration.

**Make Your File Publicly Available**

You can set the permissions on your file to public read, as explained in the [documentation][6]. This can be done from the Cloud Storage Browser or from your Node server. You can make one file public or a directory or your entire Storage database. Here's the Node code:

```js
var webmPromise = new Promise(function(resolve, reject) {
      var options = {
        destination: ('Audio/' + longLanguage + '/' + pronunciation + '/' + word + '.mp3'),
        predefinedAcl: 'publicRead',
        contentType: 'audio/' + audioType,
      };

      synthesizeParams.accept = 'audio/webm';
      var file = bucket.file('Audio/' + longLanguage + '/' + pronunciation + '/' + word + '.webm');
      textToSpeech.synthesize(synthesizeParams)
      .then(function(audio) {
        audio.pipe(file.createWriteStream(options));
      })
      .then(function() {
        console.log("webm audio file written.");
        resolve();
      })
      .catch(error => console.error(error));
    });
```

The result will look like this in your Cloud Storage Browser:

[![enter image description here][7]][7]

Anyone can then use the standard path to download your file:

```
https://storage.googleapis.com/languagetwo-cd94d.appspot.com/Audio/English/United_States-OED-0/system.mp3
```

Another way to make a file public is to use the method [makePublic()][8]. I haven't been able to get this to work, it's tricky to get the bucket and file paths right.

An interesting alternative is to use [Access Control Lists][9]. You can make a file available only to users whom you put on a list, or use `authenticatedRead` to make the file available to anyone who is logged in from a Google account. If there were an option "anyone who logged into my app using Firebase Auth" I would use this, as it would limit access to only my users.

**Build Your Own Download URL with firebaseStorageDownloadTokens** 

Several answers describe an undocumented Google Storage object property `firebaseStorageDownloadTokens`. With this you can tell Storage the token you want to use. You can generate a token with the `uuid` Node module. Four lines of code and you can build your own download URL, the same download URL you get from the console or `getDownloadURL()`. The four lines of code are:

```js
const uuidv4 = require('uuid/v4');
const uuid = uuidv4();
```

```js
metadata: { firebaseStorageDownloadTokens: uuid }
```

```js
https://firebasestorage.googleapis.com/v0/b/" + bucket.name + "/o/" + encodeURIComponent('Audio/' + longLanguage + '/' + pronunciation + '/' + word + '.webm') + "?alt=media&token=" + uuid);
```

Here's the code in context:


```js
var webmPromise = new Promise(function(resolve, reject) {
  var options = {
    destination: ('Audio/' + longLanguage + '/' + pronunciation + '/' + word + '.mp3'),
    contentType: 'audio/' + audioType,
    metadata: {
      metadata: {
        firebaseStorageDownloadTokens: uuid,
      }
    }
  };

      synthesizeParams.accept = 'audio/webm';
      var file = bucket.file('Audio/' + longLanguage + '/' + pronunciation + '/' + word + '.webm');
      textToSpeech.synthesize(synthesizeParams)
      .then(function(audio) {
        audio.pipe(file.createWriteStream(options));
      })
      .then(function() {
        resolve("https://firebasestorage.googleapis.com/v0/b/" + bucket.name + "/o/" + encodeURIComponent('Audio/' + longLanguage + '/' + pronunciation + '/' + word + '.webm') + "?alt=media&token=" + uuid);
      })
      .catch(error => console.error(error));
});
```

That's not a typo--you have to nest `firebaseStorageDownloadTokens` in double layers of `metadata:`! 

Doug Stevenson pointed out that `firebaseStorageDownloadTokens` is not an official Google Cloud Storage feature. You won't find it in any Google documentation, and there's no promise it will be in future version of `@google-cloud`. I like `firebaseStorageDownloadTokens` because it's the only way to get what I want, but it has a "smell" that it's not safe to use. 

**Why No getDownloadURL() from Node?**

As @Clinton wrote, Google should make a `file.getDownloadURL()` a method in `@google-cloud/storage` (i.e., your Node back end). I want to upload a file from Google Cloud Functions and get the token download URL.


  [1]: https://i.stack.imgur.com/f61iV.png
  [2]: https://firebase.google.com/docs/storage/web/download-files
  [3]: https://cloud.google.com/nodejs/docs/reference/storage/2.5.x/File#getSignedUrl
  [4]: https://cloud.google.com/storage/docs/gsutil/commands/signurl#options
  [5]: https://cloud.google.com/nodejs/docs/reference/storage/1.5.x/File#getSignedUrl
  [6]: https://cloud.google.com/storage/docs/access-control/making-data-public
  [7]: https://i.stack.imgur.com/V7xzE.png
  [8]: https://googleapis.dev/nodejs/storage/latest/File.html#makePublic
  [9]: https://cloud.google.com/storage/docs/access-control/lists






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

### The "Project 563584335869" error with Firebase Emulators

You might get this error with Cloud Functions in the Firebase Emulators:

```
Cloud Translation API has not been used in project 563584335869 before or it is disabled. Enable it by visiting https://console.developers.google.com/apis/api/translate.googleapis.com/overview?project=563584335869 then retry.
```

Going to that URL throws an error: `You do not have sufficient permissions to view this page`. 

I'll nominate that error message for Misleading Error Message of the Month. `563584335869` isn't your project, it's the Firebase Emulators project number. It usually means that you have a Google Cloud authorization problem. This might be solved by hooking up your service account (above) or by checking that your service account has all necessary roles, including `Firebase Admin`.

# Use Functions in your Firebase Console

You can view you functions, read the logs, get each function's URL, and delete functions from your Firebase Console.

# Testing Cloud Functions

I need to learn this.
