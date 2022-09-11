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

Create your Firestore database.

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

Open `/src/app/app.component.ts` and import three AngularFire modules.

```ts
import { Component } from '@angular/core';
import { Firestore } from '@angular/fire/firestore';

// Firebase Lite
import { collection, addDoc } from '@firebase/firestore/lite';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'GreatestComputerScientists';

  constructor() {}
}
```

## Make the HTML view

Now we'll make the view in `app.component.html`. Replace the placeholder view with:

```html
<h2>Greatest Computer Scientists</h2>

<h3>Create</h3>
<form (ngSubmit)="onCreate()">
  <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
  <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
  <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
  <button type="submit" value="Submit">Submit</button>
</form>
```

We're using an HTML form and the Angular `FormsModule`. The form is within the `<form></form>` directive.

```html
<form (ngSubmit)="onCreate()">
</form>
```

The parentheses around `ngSubmit` creates one-way data binding from the view `app.component.html` to the controller `app.component.ts`. When the `Submit` button is clicked the function `onCreate()` executes in `app.component.ts`. We'll make this function next.

Inside the form we have three text fields and a `Submit` button. The first text field has two-way data binding (parenthesis and brackets) using `ngModel` to the variable `name` in the controller. The second and third text fields bind to the variables `born` and `accomplishment`.

Clicking the button executes `ngSubmit` and the function `onCreate()`.

## Add data to Firestore

Now we'll add a handler function to write the data to database.

```ts
import { Component } from '@angular/core';
import { Firestore } from '@angular/fire/firestore';

// Firebase Lite
import { collection, addDoc } from '@firebase/firestore/lite';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'GreatestComputerScientistsLite';

  name: string | null = null;
  born: number | null = null;
  accomplishment: string | null = null;

  constructor(public firestore: Firestore) {
  }

  async onCreate() {
    try {
      const docRef = await addDoc(collection(this.firestore, 'scientists'), {
        name: this.name,
        born: this.born,
        accomplishment: this.accomplishment
      });
      console.log("Document written with ID: ", docRef.id);
    } catch (error) {
      console.error(error);
    }
  }
}
```

Enter this data in the HTML form and click `Submit`:

```
name [string]: Charles Babbage
born [number]: 1791
accomplishment [string]: Built first computer
```

Look in your Firestore console and you should see your data. Yay! Firebase is talking to your Angular app.

### Clear form fields

Notice that Charles Babbage is still in your HTML form fields. Lets's clear that data so that the forms are ready for another entry.

```ts
  async onCreate() {
    try {
      const docRef = await addDoc(collection(this.firestore, 'scientists'), {
        name: this.name,
        born: this.born,
        accomplishment: this.accomplishment
      });
      console.log("Document written with ID: ", docRef.id);
      this.name = null;
      this.born = null;
      this.accomplishment = null;
    } catch (error) {
      console.error(error);
    }
  }
```
 
## READ (once) in the view
 
Let's display the data in the HTML view. In `app.component.html` add a button that gets the data from Firestore:

```html
<h2>Greatest Computer Scientists</h2>

<h3>Create</h3>
<form (ngSubmit)="onCreate()">
    <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>

<h3>Read (once)</h3>
<form (ngSubmit)="getData()">
    <button type="submit" value="getData">Get Data</button>
</form>
```
 
## READ (once) in the controller

Add a variable `querySnapshot: any;` and a handler function:

```ts
async getData() {
    console.log("Getting data!");
    this.querySnapshot = await getDocs(collection(this.firestore, 'scientists'));
    this.querySnapshot.forEach((doc: any) => {
      console.log(`${doc.id} => ${doc.data().name}`);
    });
}
```

This should display the names of your favorite computer scientists in your console (and the ID strings of each document).

Here's the complete code:

```ts
import { Component } from '@angular/core';
import { Firestore, collectionData } from '@angular/fire/firestore';

// Firebase Lite
import { getFirestore, collection, addDoc, getDocs, updateDoc, doc } from '@firebase/firestore/lite';

interface Scientist {
  name?: string,
  born?: number,
  accomplishment?: string
};

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'GreatestComputerScientistsLite';

  name: string | null = null;
  born: number | null = null;
  accomplishment: string | null = null;
  scientists: any;
  querySnapshot: any;

  constructor(public firestore: Firestore) {
  }

  async onCreate() {
    try {
      const docRef = await addDoc(collection(this.firestore, 'scientists'), {
        name: this.name,
        born: this.born,
        accomplishment: this.accomplishment
      });
      console.log("Document written with ID: ", docRef.id);
      this.name = null;
      this.born = null;
      this.accomplishment = null;
    } catch (error) {
      console.error(error);
    }
  }

  async getData() {
    console.log("Getting data!");
    this.scientists = [];
    this.querySnapshot = await getDocs(collection(this.firestore, 'scientists'));
    this.querySnapshot.forEach((doc: any) => {
      console.log(`${doc.id} => ${doc.data().name}`);
    });
  }

}
```

### Displaying the data

Let's display this data in the HTML form. Make an array and push the scientists into the array:

```ts
  async getData() {
    console.log("Getting data!");
    this.querySnapshot = await getDocs(collection(this.firestore, 'scientists'));
    this.querySnapshot.forEach((doc: any) => {
      console.log(`${doc.id} => ${doc.data().name}`);
      this.scientists.push(doc.data());
    });
  }
```

Display the data in the HTML view:

```html
<h3>Read (once)</h3>
<form (ngSubmit)="getData()">
    <button type="submit" value="getData">Get Data</button>
</form>

<ul>
    <li *ngFor="let scientist of scientists">
        {{scientist.name}}, born {{scientist.born}}: {{scientist.accomplishment}}
    </li>
</ul>
```

OK, that works...but needs improvement. First, the TypseScript gods hate `any`. Let's make an interface (or a type, or choice).

```ts
interface Scientist {
  name?: string,
  born?: number,
  accomplishment?: string
};
```

Note that all the properties are optional. This prevents throwing an error that gets inherited from the `DocumentData` object.

Now make an array of scientists. Initialize it as an empty array.

```ts
scientists: Scientist[] = [];
```

Now we can push objects into the array:

```ts
this.scientists.push(doc.data());
```

Get rid of the initial empty array:

```ts
this.scientists = [];
```

`querySnapshot` has to remain type `any`. It seems to be type `QuerySnapshot<DocumentData>`. I have no idea how to call that as a type.

Here's the complete code so far:

```ts
import { Component } from '@angular/core';

// Firebase
import { Firestore, collectionData } from '@angular/fire/firestore';

// Firebase Lite
import { getFirestore, collection, addDoc, getDocs, updateDoc, doc } from '@firebase/firestore/lite';

interface Scientist {
  name?: string,
  born?: number,
  accomplishment?: string
};

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'GreatestComputerScientistsLite';

  name: string | null = null;
  born: number | null = null;
  accomplishment: string | null = null;

  querySnapshot: any;
  scientists: Scientist[] = [];

  constructor(public firestore: Firestore) {
  }

  async onCreate() {
    try {
      const docRef = await addDoc(collection(this.firestore, 'scientists'), {
        name: this.name,
        born: this.born,
        accomplishment: this.accomplishment
      });
      console.log("Document written with ID: ", docRef.id);
      this.name = null;
      this.born = null;
      this.accomplishment = null;
    } catch (error) {
      console.error(error);
    }
  }

  async getData() {
    this.querySnapshot = await getDocs(collection(this.firestore, 'scientists'));
    this.querySnapshot.forEach((doc: any) => {
      this.scientists.push(doc.data());
    });
  }
}
```

