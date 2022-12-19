# Cloud Functions for Firebase Tutorial

This tutorial teaches how to use [Cloud Functions for Firebase](https://firebase.google.com/docs/functions) with Firebase database, Angular, and AngularFire. You should be able to adapt this tutorial for other apps or frameworks such as React.

I use Cloud Functions to call APIs such as Google Cloud Translation and IBM Watson Speech-to-Text. You don't want to call APIs from your front end because this exposes your API keys or other credentials. You can safely call APIs from your back end, get the data, process the data, and write the data to your databases or send the data to your front end. 

The official documentation [Explore use cases](https://firebase.google.com/docs/functions/use-cases) goes into depth on other use cases for Cloud Functions. You can trigger a function when data is written to your database. You can perform intensive data processing operations or database sanitization and maintenance.

## Servers are obsolete

The "old school" way to do this is to use a server as your back end. In the 2010s running your own server became obsolete as cloud computing services became available. You can run a server in the cloud or you can skip the server and just write functions that run in the cloud. Google Cloud Functions is one provider of this service.

Cloud Functions for Firebase is a "wrapper" on Google Cloud Functions that adds funcionality if you're using Firebase databases. Cloud Functions for Firebase makes it easy to write data to Firebase databases. Cloud Functions also enables you to trigger functions when data writes to your databases.

## Firebase Local Emulator Suite
There are a couple reasons not to use Cloud Functions for Firebase. First, if you write an infinite loop you can get a big bill from Google. This can happen when you trigger a function when data is written to your database, and then the function processes the data and writes new data to the database, which triggers the function again.

Another reason not to use Cloud Functions for Firebase is that deploying an updated function to the cloud takes about ten minutes. Making a change in your code and then waiting ten minutes to see the results is frustrating.

Both of these problems are solved with the Firebase Local Emulator Suite. I can update my code and see the results in a minute. The emulator is running on your laptop so there are never any charges from Google.

# Getting Started

The official documentation [Get started](https://firebase.google.com/docs/functions/get-started) is good, except for if you want to use TypeScript. I'll assume that you're reading the official documentation and I won't go into depth repeating the information. I'll provide all the details in the section on setting up TypeScript.

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

If you don't want to install a framework you might want to initialize a Github repository.

Install AngularFire, if you're using Angular. Select `Firestore`, `Cloud Functions (callable)`, and `Cloud Storage`.

```
ng add @angular/fire
```

## Install Firebase node modules

In the project directory, install the Firebase Functions and Firebase Admin packages locally.

```
cd MyProject
npm install firebase-functions
npm install firebase-admin
```

Initialize your project.

```
firebase login
firebase init
```

Select Firestore, Functions, Storage, and Emulators.

Select your Firebase project. Accept the defaults.

You're then asked whether you want to use JavaScript or TypeScript. If you select JavaScript, skip the next section.

## Setting up TypeScript

If you select TypeScript, the next question asks if you want to use ESLint. My choice is no, as I prefer to have Visual Studio Code do linting as I write my code. 

The last question asks about installing dependencies. Say yes.

Accept the defaults for any other questions. Then you'll see:

```
✔  Firebase initialization complete!
```

Don't believe it!

You have another step to set up TypeScript. This is explained in [Use TypeScript for Cloud Functions](https://firebase.google.com/docs/functions/typescript). This documentation is 14 documents down from the [Get started](https://firebase.google.com/docs/functions/get-started) documentation. Read that documentation now or do the following steps.

Install TypeScript globally.

```
npm install -g typescript
```

### Set up emulators

You might be asked to set up the emulators in the middle of setting up TypeScript. Select `Firestore`, 'Functions`, and `Storage`.

### Update npm packages and `package.json` (optional)

You should have two `package.json` files, one for your app or front-end framework and one for your Cloud Functions.

```
MyProject/`package.json` // app or front-end framework
MyProject/functions/`package/json` // Cloud Functions
```

In your `functions` folder, check your npm packages versions:

```
npm list
```

Open `functions/package.json`. Update the version numbers of `firebase-admin`, `firebase-functions`, and `typescript`. 

Check that you have the latest versions.

```
npm outdated
```

This should return without saying anything, which indicates that you have the latest versions of all your npm packages. Run this regularly (weekly) and when it shows new versions available update your npm packages:

```
npm update
```

### Update `tsconfig.json` (optional)

Open `functions/tsconfig.json`. Add

```js
"moduleResolution": "node",
```

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

# Call and trigger Cloud Functions

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
    const original = data.text;
    const uppercase = original.toUpperCase();
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
<form (ngSubmit)="callMe(messageText)">
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

import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

*app.component.ts*
```js
import { Component } from '@angular/core';
import { getFunctions, httpsCallable, httpsCallableFromURL } from "firebase/functions";
import { initializeApp } from 'firebase/app';
import { environment } from '../environments/environment';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  firebaseConfig = environment.firebaseConfig;
  firebaseApp = initializeApp(this.firebaseConfig);

  messageText: string | null = null;
  functions = getFunctions(this.firebaseApp);

  callMe(messageText: string | null) {
    console.log("Calling Cloud Function: " + messageText);
    // const upperCaseMe = httpsCallable(this.functions, 'upperCaseMe');
    const upperCaseMe = httpsCallableFromURL(this.functions, 'http://localhost:5001/MyProject/us-central1/upperCaseMe');
    upperCaseMe({ text: messageText })
      .then((result) => {
        console.log(result.data)
      })
      .catch((error) => {
        console.error(error);
      });;
  };
}
```

### The Cloud Function's URL

We call the Cloud Function with this line:

```js
httpsCallableFromURL(this.functions, 'http://localhost:5001/MyProject/us-central1/upperCaseMe');
```

The URL is crucial. It has four parts:

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

The full URL should be available in the emulator but I can't find it.

#### Deploy to the Firebase Cloud

To call the Cloud Function in the Firebase Cloud, first deploy your Cloud Function from your `functions` folder:

```
firebase deploy --only functions:upperCaseMe
```

Then change the URL to

```js
httpsCallableFromURL(this.functions, 'https://us-central1-myprojectId.cloudfunctions.net/upperCaseMe');
```

This URL is available in your Firebase Console in your list of Functions.




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

You can see the log from the line `functions.logger.log('upperCaseMe', original, uppercase);`. You could use `console.log` instead. 

Finally the emulator throws an error: `Your function timed out after ~60s.` This seems to be a bug in the emulator. I ignore it.

## Call your Cloud Function without the URL

In `app.component.ts`, remove the comments from the line with `httpsCallable` and comment out the line with `httpsCallableFromURL`.

When I call the function now the emulator logs show that it executes correctly but my browser log shows that it returns `null`. I don't know what the problem is.



















