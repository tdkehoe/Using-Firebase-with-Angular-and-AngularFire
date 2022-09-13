# Using Firebase with Angular and AngularFire

This tutorial will make a simple Angular CRUD--CREATE, READ, UPDATE, DELETE--app that uses the Firebase Firestore cloud database, plus we'll make an OBSERVE to display realtime updates.

This project uses Angular 14, AngularFire 7.4, and Firebase Web version 9 (modular).

Here is the data we will use:

```
Charles Babbage, born 1791: Built first computer
Ada Lovelace, born 1815: Wrote first software
Howard Aiken, 1900: Built IBM's Harvard Mark I electromechanical computer
John von Neumann, born 1903: Built first general-purpose computer with memory and instructions
Alan Turing, born 1912: First theorized computers with memory and instructions, i.e., general-purpose computers
Donald Knuth, born 1938: Father of algorithm analysis
Lynn Ann Conway, born 1938: Invented generalized dynamic instruction handling
Jeff Dean, born 1968: Google's smartest computer scientist
```

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
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { provideFirestore, getFirestore } from '@angular/fire/firestore';

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

## Add data to Firestore with `add()`

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

### `set()` vs.`add()

Note that the document identifier (ID) is a string of letters and numbers. `add()` automatically generates an ID string. If you want to make the document identifier yourself you use `set()`.

Let's make a second `Submit` button for `set()`.

```html
<h2>Greatest Computer Scientists</h2>

<h3>Create (add)</h3>
<form (ngSubmit)="onCreate()">
    <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>

<h3>Create (set)</h3>
<form (ngSubmit)="onSet()">
    <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>
```

Import the `setDoc` and `doc` modules to the controller.

```ts
import { Firestore, addDoc, setDoc, doc, getDocs, collectionData, collection } from '@angular/fire/firestore';
```

We'll make a new set of variables for `set()`.

```ts
nameSet: string = '';
bornSet: number | null = null;
accomplishmentSet: string | null = null;
```

Add the handler function in the controller. Note the third parameter of `setDoc(collection())`.

```ts
  async onSet() {
    try {
      await setDoc(doc(this.firestore, 'scientists', this.nameSet), {
        name: this.nameSet,
        born: this.bornSet,
        accomplishment: this.accomplishmentSet
      });
      this.nameSet = '';
      this.bornSet = null;
      this.accomplishmentSet = null;
    } catch (error) {
      console.error(error);
    }
  }
```

A difference between `add()` and `set()` is that `set()` can update or overwrite a record. Use `Create (set)` to enter a new record:

```
Howard Aiken, 1900, Built IBM's Harvard Mark I electromechanical computer
```

I like this quote from Howard Aiken:

```
Don't worry about people stealing an idea. If it's original, you will have to ram it down their throats.
```

Now enter this data in both `Create (add)` and `Create (set)`:

```
Howard Aiken, 2000, First baby raised speaking only JavaScript
```

You should see different results. `add()` created a new record for Howard Aiken's millenial great-grandchild. `set()` updated the original Howard Aiken record.

### Coding differences between `add()` and `set()`

* Import `doc` and `setDoc` modules.
* Third parameter in `setDoc(doc())` for document identifier.
* Document identifier can't be null. Note that we initialized `nameSet` with an empty string `''`, not `null`.
 
## READ in the view
 
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

<h3>Read</h3>
<form (ngSubmit)="getData()">
    <button type="submit" value="getData">Get Data</button>
</form>
```
 
## READ in the controller

Add a variable `querySnapshot: any;` and a handler function:

```ts
async getData() {
    console.log("Getting data!");
    this.querySnapshot = await getDocs(collection(this.firestore, 'scientists'));
    this.querySnapshot.forEach((document: any) => {
      console.log(`${document.id} => ${document.data().name}`);
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
    this.querySnapshot.forEach((document: any) => {
      console.log(`${document.id} => ${document.data().name}`);
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
    this.querySnapshot.forEach((document: any) => {
      console.log(`${document.id} => ${document.data().name}`);
      this.scientists.push(document.data());
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
import { Firestore, addDoc, getDocs, collectionData, collection } from '@angular/fire/firestore';

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

  constructor(public firestore: Firestore) {}

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
    this.querySnapshot.forEach((document: any) => {
      this.scientists.push(document.data());
    });
  }
}
```

## OBSERVE in the controller

Let's get rid of that `Get Data` button. This is 2022, we're not using SQL!

Import `Observable` from `rxjs`. Make an instantiation of the `Observable` class, call it `scientist$`, and set the type as an array of `Scientist` elements: 

```ts
scientist$: Observable<Scientist[]>;
```

Here's the complete code:

```ts
import { Component } from '@angular/core';
import { Observable } from 'rxjs';

// Firebase
import { Firestore, addDoc, getDocs, collectionData, collection } from '@angular/fire/firestore';

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
  scientist$: Observable<Scientist[]>;

  constructor(public firestore: Firestore) {
    const myCollection = collection(firestore, 'scientists');
    this.scientist$ = collectionData(myCollection);
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
    this.querySnapshot.forEach((document: any) => {
      this.scientists.push(document.data());
    });
  }
}
```

### OBSERVE in the view

In the HTML view, repeat the `*ngFor` data display, with three changes. First, no button. Second, change `scientists` to `scientist$`. Third, add `| async`.

```html
<h2>Greatest Computer Scientists</h2>

<h3>Create</h3>
<form (ngSubmit)="onCreate()">
    <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>

<h3>Read</h3>
<form (ngSubmit)="getData()">
    <button type="submit" value="getData">Get Data</button>
</form>

<ul>
    <li *ngFor="let scientist of scientists">
        {{scientist.name}}, born {{scientist.born}}: {{scientist.accomplishment}}
    </li>
</ul>

<h3>Observe</h3>
<ul>
    <li *ngFor="let scientist of scientist$ | async">
        {{scientist.name}}, born {{scientist.born}}: {{scientist.accomplishment}}
    </li>
</ul>
```

You should now see the data without clicking the button, and then the same data when you click the button. Add another record and watch it change in real time. Try to make MongoDB do that!

## DELETE in the view

Now we'll add the Delete service to `app.component.html`:

```html
<h3>Delete</h3>
<form (ngSubmit)="onDelete()">
    <select name="scientist" [(ngModel)]="selection">
    <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
      {{ scientist.name }}
    </option>
  </select>

    <button type="submit" value="Submit">Delete</button>
</form>
```

This form has a `<select><option>` dropdown menu for selecting a computer scientist to delete. In this we use Angular's `*ngFor` again. Unlike the READ service we must include `[ngValue]="scientist.name"` in the Delete service. Without this your selection isn't passed back to the controller.

## DELETE in the controller

First, import the `deleteDoc` module.

```ts
import { Firestore, addDoc, doc, setDoc, getDocs, collectionData, collection, deleteDoc } from '@angular/fire/firestore';
```

Add a variable for the selection.

```ts
selection: string = '';
```

Then make a handler function.

```ts
async onDelete() {
    await deleteDoc(doc(this.firestore, 'scientists', this.selection));
    this.selection = '';
}
```

Works great...on the records that we entered with `set()`, where the document identifier is the same as the `name` field. That's a lesson in data structure: use `set()`, not `add()`, for records that may need to be deleted, and make a `name` or `identifier` field with the same document's identifier.

### DELETE by auto-generated document identifier

For documents with auto-generated document identifiers we'll have to do a query to find the document identifier.

Import the `collection`, `query`, and `where` modules.

```ts
import { Firestore, addDoc, doc, setDoc, getDocs, collectionData, collection, deleteDoc, query, where } from '@angular/fire/firestore';
```

We'll need these variables:

```ts
q: any;
querySnapshot: any;
```

And the handler function. Let's make a smelly function first:

```ts
async onDelete() {
    console.log(this.selection);
    this.q = query(collection(this.firestore, 'scientists'), where('name', '==', this.selection));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((doc: any) => {
      console.log(doc.id, ' => ', doc.data());
      deleteDoc(doc(this.firestore, 'scientists', doc.id));
    });
}
```

Do you see the problem? Look at this line:

```ts
deleteDoc(doc(this.firestore, 'scientists', doc.id));
```

`doc` is both a Firestore module and the elements in the array being iterated. I've sent feedback to the Firebase team suggesting renaming the module `document` (to go with `collection`).

We can't rename the Firestore module so we'll have change the name of the elements in the array.

```ts
async onDelete() {
    console.log(this.selection);
    this.q = query(collection(this.firestore, 'scientists'), where('name', '==', this.selection));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((document: any) => {
      console.log(document.id, ' => ', document.data());
      deleteDoc(doc(this.firestore, 'scientists', document.id));
    });
}
```

This will delete all documents with the selected name. My database had several `Charles Babbage` documents. Now they're all gone. :-)

## UPDATE in the view

```html
<h3>Update</h3>
<form (ngSubmit)="onSelect()">
    <select name="scientist" [(ngModel)]="selectionUpdate">
        <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
            {{ scientist.name }}
        </option>
    </select>

    <input type="text" [(ngModel)]="bornUpdate" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishmentUpdate" name="accomplishment" placeholder="Accomplishment">

    <button type="submit" value="Submit">Select</button>
</form>

<form (ngSubmit)="onUpdate()">
    <button type="submit" value="Update">Update</button>
</form>
```

Note that there are two buttons. `Submit` pulls up the data after you select a computer scientist. Then you change the data and click `Update`.

## UPDATE in the controller

Import the `updateDoc` module.

```ts
import { Firestore, addDoc, doc, setDoc, getDocs, collectionData, collection, deleteDoc, query, where, updateDoc } from '@angular/fire/firestore';
```

Make some more variables:

```ts
nameUpdate: string = '';
bornUpdate: number | null = null;
accomplishmentUpdate: string | null = null;
selectionUpdate: string = '';
```

Add the handler functions:

```ts
async onSelect() {
    console.log(this.selectionUpdate);
    this.q = query(collection(this.firestore, 'scientists'), where('name', '==', this.selectionUpdate));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((document: any) => {
      console.log(document.id, ' => ', document.data());
      this.nameUpdate = document.data().name;
      this.bornUpdate = document.data().born;
      this.accomplishmentUpdate = document.data().accomplishment;
    });
}
  
async onUpdate() {
    this.q = query(collection(this.firestore, 'scientists'), where('name', '==', this.nameUpdate));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((document: any) => {
      console.log(document.id, ' => ', document.data());
      this.nameUpdate = document.id;
    });
    await updateDoc(doc(this.firestore, 'scientists', this.nameUpdate), {
      born: this.bornUpdate,
    });
    this.bornUpdate = null;
    this.accomplishmentUpdate = null;
}
```

We have two handler functions, for the two buttons. `onSelect()` takes the name that the user selects and gets the data from the document. The user can then edit the data. The user clicks `onUpdate()` which finds the document by the `name` field and gets the document identifier. The document identifier is then used in `updateDoc()`.

OK, I'm not proud of this `UPDATE` function. Because some documents have their document identifier matching the `name` field but other documents have a random string for their document identifier it's not possible to update the `name` field. In a real app the document identifiers would be standardized one way or the other. The two buttons are a poor user interface. In a real app the `Submit` button would be gone, with realtime data filling the view fields. `UPDATE` may seem like a simple CRUD function but it's actually the most complex because it has to let the user select a document, update one or more fields, without changing the document identifier.

## `any` Type

A frustrating part of using Firebase with Typescript is handling the returning collections and documents without using the data type `any`. I found these variables typed as `any` in our controller:

* `querySnapshot` receives the collection `scientists` in the `getDocs()` data READ once function
* `q` receives the collection that results from a query, i.e., a subset of the complete collection in the database
* `document` is an document in an collection. `document.data` is type `Scientist` but `document` has additional properties.

Firestore has a [data converter feature to make custom objects](https://firebase.google.com/docs/firestore/manage-data/add-data#custom_objects) but I don't see how this will get rid of `any` here.

