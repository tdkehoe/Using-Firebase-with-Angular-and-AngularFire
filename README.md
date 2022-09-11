# Using Firebase with Angular and AngularFire

This tutorial will make a simple Angular app that uses the Firebase Firestore cloud database. We'll start with Firestore Lite, which only handles CRUD--Create, Read, Update, Delete--operations and is up to 80% lighter, and then we'll do realtime updates.

This project uses Angular 14, AngularFire 7.4, and Firebase Web version 9 (modular). 

## Create a new project

In your terminal:

```bash
npm install -g @angular/cli
ng new GreatestComputerScientists
cd GreatestComputerScientists
```

The Angular CLI's `new` command will set up the latest Angular build in a new project structure. Accept the defaults (no routing, CSS). Start the server:

```bash
ng serve -o
```

Your browser should open to `localhost:4200`. You should see the Angular default homepage.

## Install AngularFire and Firebase

Open another tab in your terminal and install AngularFire and Firebase from `npm`.

```bash
ng add @angular/fire
```

Deselect `ng deploy -- hosting` and select `Firestore`. This project won't use any other Firebase features.

It will ask you for your email address associated with your Firebase account. Then it will ask you to associate a Firebase project. Select `[CREATE NEW PROJECT]`. Call it `Greatest Computer Scientists`.

If this doesn't work, open your Firebase console and make a new project. Call it `Greatest Computer Scientists`. Skip the Google Analytics.

## Add Firebase config to `environments` variable

In your [Firebase Console](https://console.firebase.google.com), under `Get started by adding Firebase to your app` select the web app `</>` icon. Register your app, again calling it `Greatest Computer Scientists`. You won't need `Firebase Hosting`, we'll just run the app locally.

Under `Add Firebase SDK` select `Use npm`.

Return to your terminal and install Firebase in your new project.

```bash
npm install firebase
```

Then copy and paste the config values provided to your `environment.ts` file:

```ts
export const environment = {
  production: false,
  firebaseConfig: {
    apiKey: '<your-key>',
    authDomain: '<your-project-authdomain>',
    projectId: '<your-project-id>',
    storageBucket: '<your-storage-bucket>',
    messagingSenderId: '<your-messaging-sender-id>',
    appId: '<your-app-id>',
    measurementId: '<your-measurement-id>'
  }
};
```

Don't copy and paste the whole window that the Firebase console provides. Check that it looks like the above code. A `=` needs to be changed to `:` and a `;` needs to be dropped. Then check that your browser is still showing the demo app.

`Add Firebase SDK` also tells you to do several other things:

```
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Initialize Firebase
const app = initializeApp(firebaseConfig);
```

We'll use AngularFire instead of these items. Click `Continue to console`.

## Setup `@NgModule` for the `AngularFireModule`

Open `/src/app/app.module.ts`, import the `environment` module and the `FormsModule` module. Then import a bunch of AngularFire modules. Lastly, import the FormsModule and two AngularFire functions to initialize Firebase and get Firestore.

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { environment } from '../environments/environment'; // access firebaseConfig

// Angular
import { FormsModule } from '@angular/forms';

// AngularFire
import { provideFirebaseApp, getApp, initializeApp } from '@angular/fire/app';
import { getFirestore, provideFirestore } from '@angular/fire/firestore';


@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    provideFirebaseApp(() => initializeApp(environment.firebaseConfig)),
    provideFirestore(() => getFirestore()),
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Keep an eye on the browser. If the homepage crashes, go back and see what's wrong.

## Inject `AngularFirestore` into Component Controller

Open `/src/app/app.component.ts` and import the three AngularFire modules. In the `constructor` make a public variable for `Firestore`. This needs to be public so we can use Firestore in functions, not just in the constructor.

```ts
import { Component } from '@angular/core';
import { Firestore, collectionData, collection } from '@angular/fire/firestore';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'GreatestComputerScientists';

  constructor(public firestore: Firestore) {}
}
```

## Make a `Scientist` interface

Make a scientist object:

```ts
interface Scientist {
  name?: string,
  born?: number,
  accomplishment?: string
};
```

Note that all three fields are optional. Firestore collections don't convert into Typescript arrays unless the properties are optional. It has to do with the inherited `DocumentData` type.

You can make a `type` instead of an interface, it doesn't matter.

## Add data to Firestore

