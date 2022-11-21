# Using Firebase with Angular and AngularFire

- [Using Firebase with Angular and AngularFire](#using-firebase-with-angular-and-angularfire)
    - [Collections and documents](#collections-and-documents)
  - [Create a new project](#create-a-new-project)
  - [Install AngularFire and Firebase](#install-angularfire-and-firebase)
  - [Add Firebase config to `environments` variable](#add-firebase-config-to-environments-variable)
  - [Setup `@NgModule` for the `AngularFireModule`](#setup-ngmodule-for-the-angularfiremodule)
  - [Inject `AngularFirestore` into Component Controller](#inject-angularfirestore-into-component-controller)
  - [Make the HTML view](#make-the-html-view)
  - [Add data to Firestore with `add()`](#add-data-to-firestore-with-add)
    - [Clear form fields](#clear-form-fields)
    - [`set()` vs. `add()`](#set-vs-add)
    - [Coding differences between `add()` and `set()`](#coding-differences-between-add-and-set)
    - [`set({}, {merge: true})`](#set-merge-true)
  - [CREATE collections](#create-collections)
  - [READ document in the view](#read-document-in-the-view)
  - [READ document in the controller](#read-document-in-the-controller)
  - [READ nested document](#read-nested-document)
  - [READ collection in the controller](#read-collection-in-the-controller)
    - [Displaying the data](#displaying-the-data)
    - [Filter with `query` and `where`](#filter-with-query-and-where)
    - [Order and limit data](#order-and-limit-data)
    - [Using the Firebase data converter to make custom objects](#using-the-firebase-data-converter-to-make-custom-objects)
  - [OBSERVE a collection listener in the controller](#observe-a-collection-listener-in-the-controller)
    - [OBSERVE a collection listener in the view](#observe-a-collection-listener-in-the-view)
    - [OBSERVE a document listener in the controller](#observe-a-document-listener-in-the-controller)
    - [OBSERVE a document listener in the view](#observe-a-document-listener-in-the-view)
    - [Detach a listener](#detach-a-listener)
  - [DELETE in the view](#delete-in-the-view)
  - [DELETE in the controller](#delete-in-the-controller)
    - [DELETE by auto-generated document identifier](#delete-by-auto-generated-document-identifier)
    - [DELETE fields, collections, and subcollections](#delete-fields-collections-and-subcollections)
  - [UPDATE in the view](#update-in-the-view)
  - [UPDATE in the controller](#update-in-the-controller)
    - [`update()` vs. `set({}, {merge: true})`](#update-vs-set-merge-true)
  - [UPDATE collections](#update-collections)
  - [To Do: Stuff I don't understand](#to-do-stuff-i-dont-understand)
    - [`any` ype for returned collections and documents, $events, snapshot](#any-ype-for-returned-collections-and-documents-events-snapshot)
    - [`unsubscribe()` from collections listener](#unsubscribe-from-collections-listener)
  - [Complete finished code](#complete-finished-code)
    - [`environments/environment.ts`](#environmentsenvironmentts)
    - [`app.module.ts`](#appmodulets)
    - [`app.component.html`](#appcomponenthtml)
    - [`app.component.ts`](#appcomponentts)

This tutorial will make a simple Angular CRUD (CREATE, READ, UPDATE, DELETE) app that uses Google's Firebase Firestore cloud database, plus we'll make an OBSERVE to display realtime updates.

This project uses Angular 14, AngularFire 7.4, and Firestore Web version 9 (modular). 

Firestore Web version 9 is a big advance. Load time is reduced by as much as 80%. I like that the documentation is written with `async await` instead of promises.

I assume that you know the basics of Angular (nothing advanced is required). No CSS or styling is used, to make the code easier to understand.

### Collections and documents

I assume that you know the [basics](https://firebase.google.com/docs/firestore) of Firebase's Firestore cloud database. In particular, this tutorial talks about `collections` and `documents`. This is essential. A `collection` is pretty much an array and a `document` is pretty much an object, just like in JavaScript. Firestore is structured with collections of documents. A document can contain a collection (a.k.a. a sub-collection), which contains further documents. This `collection`-`document`-`collection`-`document` pattern is maintained. 

You'll see that some of the CRUD operations come in pairs: one for a collection and one for a document. You can READ a collection with `getDoc()` or READ a collection with `getDocs()`. You can OBSERVE a collection or OBSERVE a document. The other three CRUD operations (CREATE, UPDATE, DELETE) are only for documents. You, as admin, can CREATE and DELETE collections from the Firebase console but users can't do these operations from a web app.

I expect that you'll read this tutorial with a second window open to the [Firebase documentation](https://firebase.google.com/docs/firestore) and the [AngularFire documentation](https://github.com/angular/angularfire). I'll try to let you know which page of the Firebase documentation to open for each section of this tutorial. This stuff changes, especially AngularFire. Make a pull request if something in this tutorial is out of date.

Here is the data (documents) we will use:

```
Charles Babbage, born 1791: Built first computer
Ada Lovelace, born 1815: Wrote first software
Howard Aiken, 1900: Built IBM's Harvard Mark I electromechanical computer
John von Neumann, born 1903: Built first general-purpose computer with memory and instructions
Grace Hopper, born 1906: Devised the first machine-independent programming language.
Alan Turing, born 1912: First theorized computers with memory and instructions, i.e., general-purpose computers
Donald Knuth, born 1938: Father of algorithm analysis
Lynn Ann Conway, born 1938: Invented generalized dynamic instruction handling
Shafi Goldwasser, born 1958: Crypography and blockchain
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

Open the Firestore [Get started](https://firebase.google.com/docs/firestore/quickstart) section.

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

Open the [AngularFire documentation](https://github.com/angular/angularfire) for this section.

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

// AngularFire
import { collection, addDoc } from '@angular/fire/firestore';

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

Open the [Add Data](https://firebase.google.com/docs/firestore/quickstart#add_data) section of the Firestore documentation.

Now we'll add a handler function to write the data to database.

```ts
async onCreate() {
    try {
      const docRef = await addDoc(collection(this.firestore, 'Scientists'), {
        name: this.name,
        born: this.born,
        accomplishment: this.accomplishment
      });
      console.log("Document written with ID: ", docRef.id);
    } catch (error) {
      console.error(error);
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
      const docRef = await addDoc(collection(this.firestore, 'Scientists'), {
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

### `set()` vs. `add()`

Open the [Add data](https://firebase.google.com/docs/firestore/manage-data/add-data) section of the Firestore documentation.

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
      await setDoc(doc(this.firestore, 'Scientists', this.nameSet), {
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
 
### `set({}, {merge: true})`

`set()` will overwrite a document or create it if it doesn't exist yet. Another option is to add the `{merge: true}`:

```js
set({data: 12345}, {merge: true})
```

This will update fields in the document or create it if it doesn't exists. In contrast, `update()` will update fields but will fail if the document doesn't exist. We'll go over this again in the `update()` section.
 
## CREATE collections

You, as admin, can create collections in the Firebase console or the CLI. Your users can't create collections from a web app.
 
## READ document in the view
 
Open the [Get data once](https://firebase.google.com/docs/firestore/query-data/get-data) section of the Firestore documentation.
 
In `app.component.html` add a button that allows you to select a computer scientist by name and then click a button to display their birthyear and accomplishment. Don't try to understand how the list of computer scientists gets into the `<select><option>`, we'll get to that in the `OBSERVE` section.

```html
<h2>Greatest Computer Scientists</h2>

<h3>Create</h3>
<form (ngSubmit)="onCreate()">
    <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>

<h3>Read (one document, once)</h3>

<form (ngSubmit)="getDocument()">
    <select name="scientist" [(ngModel)]="documentID">
        <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
            {{ scientist.name }}
        </option>
    </select>

    <button type="submit" value="Submit">Get Document</button>
</form>

<div *ngIf="singleDoc.name">{{ singleDoc.name }}, born {{ singleDoc.born }}: {{ singleDoc.accomplishment }}</div>
```

We're using `*ngIf` to hide the result before the user selects a scientist.

## READ document in the controller

Import the `doc` and `getDoc` modules.

```ts
import { doc, getDoc } from '@angular/fire/firestore';
```

Make some variables.

```ts
documentID: string = '';
docSnap: any;
docSnapName: string = '';
docSnapBorn: number | null = null;
docSnapAccomplishment: string = '';
```

Make the handler function.

```ts
async getDocument() {
    this.docSnap = await getDoc(doc(this.firestore, 'Scientists', this.documentID));
    this.docSnapName = this.docSnap.data().name;
    this.docSnapBorn = this.docSnap.data().born;
    this.documentID = this.docSnap.data().accomplishment;
}
```

Couldn't be simpler! Well, it could. Let's make a variable to hold the data:

```ts
singleDoc: Scientist = {
    name: null,
    born: null,
    accomplishment: null
}
```

Simplfy the handler function:

```ts
async getDocument() {
    this.docSnap = await getDoc(doc(this.firestore, 'Scientists', this.documentID));
    this.singleDoc = this.docSnap.data();
}
```

And display the data:

```html
{{ singleDoc.name }}, born {{ singleDoc.born }}: {{ singleDoc.accomplishment }}
```

## READ nested document

In your Firebase console, make a new document in your `Scientists` collection. Call it `nest`, with one field `name: nest`. In the `nest` document make a sub-collection `Nested`. Note that we're using Uppercase for collections and lowercase for documents with subcollections.

Add one document to your `Nested` subcollection from the Firebase console.

Add some variables to your controller:

```ts
nestedScientist$: Observable<Scientist[]>;
nestedDocumentID: string = '';

nestedDoc: Scientist = {
    name: null,
    born: null,
    accomplishment: null
}
```

Add a handler function to your controller:

```ts
async getNested() {
    try {
      this.docSnap = await getDoc(doc(this.firestore, 'Scientists/nest/Nested', this.nestedDocumentID));
      this.nestedDoc = this.docSnap.data();
    } catch (error) {
      console.error(error);
    }
}
```

Put the nested document in the view:

```html
<h3 ng>Read (nested document, once)</h3>

<form (ngSubmit)="getNested()">
    <select name="nested" [(ngModel)]="nestedDocumentID">
        <option *ngFor="let scientist of nestedScientist$ | async" [ngValue]="scientist.name">
            {{ scientist.name }}
        </option>
    </select>

    <button type="submit" value="Submit">Get Nested</button>
</form>

<div *ngIf="nestedDoc.name">{{ nestedDoc.name }}, born {{ nestedDoc.born }}: {{ nestedDoc.accomplishment }}</div>
```

You should be able to select a nested computer scientist and then see the results in the view. We won't make a CREATE, UPDATE, or DELETE for the nested data.

## READ collection in the controller

Add a variable 

```ts
`querySnapshot: any;` 
```

and a handler function:

```ts
async getData() {
    console.log("Getting data!");
    this.querySnapshot = await getDocs(collection(this.firestore, 'Scientists'));
    this.querySnapshot.forEach((document: any) => {
      console.log(`${document.id} => ${document.data().name}`);
    });
}
```

The first line clears any old data in the view. This should display the names of your favorite computer scientists in your console.

Let's add `query where` to filter the results

### Displaying the data

Let's display this data in the HTML form. Make an array and push the scientists into the array:

```ts
  async getData() {
    this.querySnapshot = await getDocs(collection(this.firestore, 'Scientists'));
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

OK, that works...but needs improvement. First, the TypseScript gods hate `any`. Let's make an `interface` (or a `type`, your choice).

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

Also, let's clear any old data from the view:

```ts
    this.scientists = []; // clear view
```

### Filter with `query` and `where`

Make a variable:

```ts
whenBorn: string = '1700';
```

This should be a number but it only works as a string. Sometimes TypeScript is baffling.

And make a query in the handler function:

```ts
async getData() {
    this.scientists = []; // clear view
    this.q = query(collection(this.firestore, 'Scientists'), where('born', '>=', this.whenBorn));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((docElement: any) => {
      this.scientists.push(docElement.data());
    });
}
```

The user can now filter results to show only computer scientists born after 1700, 1800, or 1900.

### Order and limit data

By default the data is ordered by the document identifier. You can [order your data differently or limit the number of documents](https://firebase.google.com/docs/firestore/query-data/order-limit-data) returned.

### Using the Firebase data converter to make custom objects

I can't get the Firestore [data converter](https://firebase.google.com/docs/firestore/query-data/get-data#custom_objects) to work. It should convert downloaded documents into custom objects, e.g., `Scientist`, or convert objects to be uploaded to Firestore into a specific collection. 

## OBSERVE a collection listener in the controller

Open the [Listen for realtime updates](https://firebase.google.com/docs/firestore/query-data/listen) section of the Firestore documentation.

Let's get rid of that `Get Data` button. This is 2022, we're not using SQL!

Import `Observable` from `rxjs`. Make an instantiation of the `Observable` class, call it `scientist$`, and set the type as an array of `Scientist` elements: 

```ts
scientist$: Observable<Scientist[]>;
```

The observer is one line in the `constructor`:

```ts
 constructor(public firestore: Firestore) {
    this.scientist$ = collectionData(collection(firestore, 'Scientists'));
  }
```

That's it. Now `scientist$` will always mirror the database. We have the observer in the `constructor` so that it starts when the page loads (`ngOnInit` would do more or less the same thing). An observer could instead go in a function to start after an event.

### OBSERVE a collection listener in the view

In the HTML view, repeat the `*ngFor` data display, with three changes. First, no button. Second, change `scientists` to `scientist$`. Third, add the pipe `| async`.

```html
<h3>Observe</h3>
<ul>
    <li *ngFor="let scientist of scientist$ | async">
        {{scientist.name}}, born {{scientist.born}}: {{scientist.accomplishment}}
    </li>
</ul>
```

You should now see the data without clicking the button, and then the same data when you click the button. Add another record and watch it change in real time. Try to make MongoDB do that!

### OBSERVE a document listener in the controller

We can also observe a single document. In the controller, make some variables:

```ts
charle$: Scientist  = {
    name: null,
    born: null,
    accomplishment: null
  };
unsubCharle$: any;
```

Then make the listener in the `constructor`:

```ts
constructor(public firestore: Firestore) {
    this.scientist$ = collectionData(collection(firestore, 'Scientists')); // collection listenr
    this.unsubCharle$ = onSnapshot(doc(firestore, 'Scientists', 'Charles Babbage'), (snapshot: any) => { // document listener
        this.charle$.name = snapshot.data().name;
        this.charle$.born = snapshot.data().born;
        this.charle$.accomplishment = snapshot.data().accomplishment;
    });
}
```

This makes two observers, the collection listener and the document listener.

### OBSERVE a document listener in the view

```html
<h3>Observe (single document, 'Charles Babbage')</h3>
<div *ngIf="charle$.name">{{ charle$.name }}, born {{ charle$.born }}: {{ charle$.accomplishment}}</div>
```

### Detach a listener

When you no longer need to observe a collection, [detach the listener](https://firebase.google.com/docs/firestore/query-data/listen#detach_a_listener) to reduce your bandwidth.

Detaching the document listener is easy:

```html
<form (ngSubmit)="detachListener()">
    <button type="submit" value="detachListener">Detach Listener</button>
</form>
```

```ts
async detachListener() {
    console.log("Detaching listener.");
    this.unsubCharle$();
}
```

I can't figure out how to detach the collection listener.

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
    await deleteDoc(doc(this.firestore, 'Scientists', this.selection));
    this.selection = '';
}
```

Works great...on the records that we entered with `set()`, where the document identifier is the same as the `name` field. That's a lesson in data structure: use `set()`, not `add()`, for records that may need to be deleted, and make a `name` or `identifier` field with the same document's identifier.

### DELETE by auto-generated document identifier

Open [Perform simple and compound queries in Cloud Firestore](https://firebase.google.com/docs/firestore/query-data/queries) in the Firestore documentation. 

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

These variables are type `any` because they will handle collections returned from Firestore. I don't know what type a collection is.

Now we'll make the handler function. Let's make a smelly function first:

```ts
async onDelete() {
    console.log(this.selection);
    this.q = query(collection(this.firestore, 'Scientists'), where('name', '==', this.selection));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((doc: any) => {
      console.log(doc.id, ' => ', doc.data());
      deleteDoc(doc(this.firestore, 'Scientists', doc.id));
    });
}
```

Do you see the problem? Look at this line:

```ts
deleteDoc(doc(this.firestore, 'Scientists', doc.id));
```

`doc` is both a Firestore module and the elements in the array being iterated. We can use an alias for the module `doc`:

```ts
import { doc as whatsUpDoc } from '@angular/fire/firestore';
```

but I can't think of a better name for the module than `doc`. We can't use `document` because that's a [JavaScript keyword](https://developer.mozilla.org/en-US/docs/Web/API/Document). I suspect that the Firebase team had a discussion about this.

We'll have change the name of the elements in the array from `doc` to `docElement`.

```ts
async onDelete() {
    console.log(this.selection);
    this.q = query(collection(this.firestore, 'Scientists'), where('name', '==', this.selection));
    this.querySnapshot = await getDocs(this.q);
    this.querySnapshot.forEach((docElement: any) => {
      console.log(docElement.id, ' => ', docElement.data());
      deleteDoc(doc(this.firestore, 'Scientists', document.id));
    });
}
```

This will delete all documents with the selected name. My database had several `Charles Babbage` documents. Now they're all gone. :-)

More about filtering queries with [where()](https://firebase.google.com/docs/firestore/query-data/queries).

### DELETE fields, collections, and subcollections

Our `DELETE` functions deletes documents. 

To delete fields in documents use [deleteField()](https://firebase.google.com/docs/firestore/manage-data/delete-data#fields). 

If a field in a document is a collection, i.e., a subcollection, the documents in those subcollections won't be deleted with the parent document. You'll have to search for and delete each document in a subcollection, or delete the subcollection from the Firebase Console.

You, as admin, can delete collections in the Firebase console or the CLI. Your users can't delete collections from a web app.

## UPDATE in the view

```html
<h3>Update</h3>
<form (ngSubmit)="onUpdate()">
    <select (change)="onSelect($event)">
        <option>Select scientist</option>
        <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
            {{ scientist.name }}
        </option>
    </select>

    <input type="text" [(ngModel)]="bornUpdate" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishmentUpdate" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Update">Update</button>
</form>
```

The `<select><option>` menu uses [Angular event binding](https://angular.io/guide/event-binding-concepts). `(change)` looks for a change in the selected option, from `Select scientist` to a computer scientist. This fires the `onSelect()` handler function in the controller. `$event` passes the selected value to the handler function.

The `*ngFor` iterates through the observer's data `scientist$` to display a realtime view of the computer scientists in the database.

Two text `input` fields allow changing the computer scientist's year of birth and accomplishment (not their name).

The button fires a second handler function `onUpdate()` to update the database. 

## UPDATE in the controller

Open [Update a document](https://firebase.google.com/docs/firestore/manage-data/add-data#update-data) in the Firestore documentation.

Import the `updateDoc` module.

```ts
import { Firestore, addDoc, doc, setDoc, getDocs, collectionData, collection, deleteDoc, query, where, updateDoc } from '@angular/fire/firestore';
```

Make some more variables:

```ts
nameUpdate: string = '';
bornUpdate: number | null = null;
accomplishmentUpdate: string | null = null;
```

We can't collapse these three variables into a `Scientist` type object because the `name` property can't be `null`.

Add the handler functions:

```ts
// UPDATE
async onSelect(event: any) { // type Event doesn't work despite https://angular.io/guide/event-binding-concepts
    this.querySnapshot = await getDocs(query(collection(this.firestore, 'Scientists'), where('name', '==', event.target.value))); // query the database
    this.querySnapshot.forEach((docElement: any) => { // itereate through the collection
      this.nameUpdate = docElement.data().name; // transfer to local variables
      this.bornUpdate = docElement.data().born;
      this.accomplishmentUpdate = docElement.data().accomplishment;
    });
}

// UPDATE
async onUpdate() {
    this.querySnapshot = await getDocs(query(collection(this.firestore, 'Scientists'), where('name', '==', this.nameUpdate))); // find the document by the name property instead of the document identifier because we're using both autogenerated document identifiers and custom document identifiers
    this.querySnapshot.forEach((docElement: any) => { // iterate through the collection to find the document identifier for the selected document
      this.nameUpdate = docElement.id; // put the document identifier in a local variable
    });
    await updateDoc(doc(this.firestore, 'Scientists', this.nameUpdate), { // update the database
      born: this.bornUpdate,
      accomplishment: this.accomplishmentUpdate,
    });
    this.bornUpdate = null; // clear form field
    this.accomplishmentUpdate = null; // clear form field
}
```

We have two handler functions, making UPDATE more complex than the other CRUD features. `onSelect()` fires when the user selects a compputer scientist, then quieries the database to fill in the year and the accomplishment. The user then updates one or both fields and clicks the button that fires `onUpdate()`. This handler fucntion queries the database to find the document by the `name` property and return the document identifier. We could eliminate this code if we used a data structure in which the document identifier always matched the `name` property.

When we have the document identifier we then run `updateDoc()` to update the database.

### `update()` vs. `set({}, {merge: true})`

As we learned earlier, `set()` will overwrite a document or create it if it doesn't exist yet. But we can append `{merge: true}` to `set()`:

```js
set({data: 12345}, {merge: true})
```

This will update fields in the document or create it if it doesn't exists. In contrast, `update()` will update fields but will fail if the document doesn't exist. You might call this "updateOrCreate()".

## UPDATE collections

Can't do that from anywhere.

## To Do: Stuff I don't understand

There's a few things that I either don't understand of Firebase can't do.

### `any` ype for returned collections and documents, $events, snapshot

The TypeScript gods hate it when we use `any` but collections and documents returned from Firestore are type `any`, as far as I know. You send data to Firestore as a `Scientist` custom type but it comes back as a document in which the data is now `document.data`, i.e., Firestore puts a container around your data. The other stuff Firestore returns, including `$event` and `snapshot`, also have to be typed `any`. (I tried typing `$event` as `Event` but this threw an error.)

Maybe Firestore could provide a module with interfaces or types, `Collection` and `Document`, that we could use instead of `any`?

Firestore has a [data converter feature to make custom objects](https://firebase.google.com/docs/firestore/manage-data/add-data#custom_objects) but I don't see how this will get rid of `any` here.

### `unsubscribe()` from collections listener

I couldn't get this code to work.

## Complete finished code

Please send pull requests if you find mistakes.

This Angular app has four modified files.

### `environments/environment.ts`

```ts
export const environment = {
  production: false,
  firebaseConfig: {
    apiKey: "abc123",
    authDomain: "myCRUDyApp.firebaseapp.com",
    projectId: "myCRUDyApp",
    storageBucket: "myCRUDyApp.appspot.com",
    messagingSenderId: "12345",
    appId: "1:12345:web:abcde"
  }
};
```

### `app.module.ts`

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

### `app.component.html`

```ts
<h2>Greatest Computer Scientists</h2>

<h3>Create document (add)</h3>
<form (ngSubmit)="onCreate()">
    <input type="text" [(ngModel)]="name" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="born" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishment" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>

<h3>Create document (set)</h3>
<form (ngSubmit)="onSet()">
    <input type="text" [(ngModel)]="nameSet" name="name" placeholder="Name" required>
    <input type="text" [(ngModel)]="bornSet" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishmentSet" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Submit">Submit</button>
</form>

<h3 ng>Read (document, once)</h3>

<form (ngSubmit)="getDocument()">
    <select name="scientist" [(ngModel)]="documentID">
        <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
            {{ scientist.name }}
        </option>
    </select>

    <button type="submit" value="Submit">Get Document</button>
</form>

<div *ngIf="singleDoc.name">{{ singleDoc.name }}, born {{ singleDoc.born }}: {{ singleDoc.accomplishment }}</div>

<h3>Read (collection, once)</h3>

<form (ngSubmit)="getData()">
    <button type="submit" value="getData">Get Collection</button>
</form>

<ul>
    <li *ngFor="let scientist of scientists">
        {{scientist.name}}, born {{scientist.born}}: {{scientist.accomplishment}}
    </li>
</ul>

Show only computer scientists born after:
<select name="whenBorn" [(ngModel)]="whenBorn">
    <option>1700</option>
    <option>1800</option>
    <option>1900</option>
</select> (filter documents with <code>query</code> and <code>where</code>)

<h3>Observe (collection)</h3>
<ul>
    <li *ngFor="let scientist of scientist$ | async">
        {{scientist.name}}, born {{scientist.born}}: {{scientist.accomplishment}}
    </li>
</ul>

<form (ngSubmit)="detachListener()">
    <button type="submit" value="detachListener">Detach Listener</button>
</form>

<h3>Observe (document, 'Charles Babbage')</h3>
<div *ngIf="charle$.name">{{ charle$.name }}, born {{ charle$.born }}: {{ charle$.accomplishment}}</div>

<h3>Update document</h3>
<form (ngSubmit)="onUpdate()">
    <select (change)="onSelect($event)">
        <option>Select scientist</option>
        <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
            {{ scientist.name }}
        </option>
    </select>

    <input type="text" [(ngModel)]="bornUpdate" name="born" placeholder="Year born">
    <input type="text" [(ngModel)]="accomplishmentUpdate" name="accomplishment" placeholder="Accomplishment">
    <button type="submit" value="Update">Update</button>
</form>

<h3>Delete document</h3>
<form (ngSubmit)="onDelete()">
    <select name="deleteScientist" [(ngModel)]="selection">
        <option *ngFor="let scientist of scientist$ | async" [ngValue]="scientist.name">
            {{scientist.name}}
        </option>
    </select>

    <button type="submit" value="Submit">Delete</button>
</form>
```

### `app.component.ts`

```ts
import { Component, OnInit } from '@angular/core';
import { Observable } from 'rxjs';

// Firebase
import { Firestore, doc, addDoc, setDoc, getDoc, getDocs, collectionData, collection, deleteDoc, query, where, updateDoc, onSnapshot } from '@angular/fire/firestore';

interface Scientist {
  name?: string | null,
  born?: number | null,
  accomplishment?: string | null
};

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'GreatestComputerScientistsLite';

  //CREATE add()
  name: string | null = null;
  born: number | null = null;
  accomplishment: string | null = null;

  // CREATE set()
  nameSet: string = '';
  bornSet: number | null = null;
  accomplishmentSet: string | null = null;

  // UPDATE
  nameUpdate: string = '';
  bornUpdate: number | null = null;
  accomplishmentUpdate: string | null = null;

  querySnapshot: any;

  // READ collecion
  scientists: Scientist[] = [];
  // OBSERVE
  scientist$: Observable<Scientist[]>;
  charle$: Scientist  = {
    name: null,
    born: null,
    accomplishment: null
  };
  unsubCharle$: any;

  selection: string = '';
  documentID: string = '';
  docSnap: any;

  // READ document
  singleDoc: Scientist = {
    name: null,
    born: null,
    accomplishment: null
  }


  setDoc: Scientist = {
    name: null,
    born: null,
    accomplishment: null
  }

  deleteID: string = '';
  deleteIDarray: string[] = [];

  whenBorn: string = '1700';

  constructor(public firestore: Firestore) {
    // OBSERVER
    this.scientist$ = collectionData(collection(firestore, 'Scientists')); // collection listener
    this.unsubCharle$ = onSnapshot(doc(firestore, 'Scientists', 'Charles Babbage'), (snapshot: any) => { // document listener
        this.charle$.name = snapshot.data().name;
        this.charle$.born = snapshot.data().born;
        this.charle$.accomplishment = snapshot.data().accomplishment;
    });
  }

  // OBSERVER document listener
  detachListener() {
    this.unsubCharle$();
  }

  // CREATE add()
  async onCreate() {
    try {
      const docRef = await addDoc(collection(this.firestore, 'Scientists'), {
        name: this.name,
        born: this.born,
        accomplishment: this.accomplishment
      });
      this.name = null;
      this.born = null;
      this.accomplishment = null;
    } catch (error) {
      console.error(error);
    }
  }

  // CREATE set()
  async onSet() {
    try {
      await setDoc(doc(this.firestore, 'Scientists', this.nameSet), {
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

  // READ document
  async getDocument() {
    try {
      this.docSnap = await getDoc(doc(this.firestore, 'Scientists', this.documentID));
      this.singleDoc = this.docSnap.data();
    } catch (error) {
      console.error(error);
    }
  }

  // READ collection
  async getData() {
    try {
      this.scientists = []; // clear view
      this.querySnapshot = await getDocs(query(collection(this.firestore, 'Scientists'), where('born', '>=', this.whenBorn)));
      this.querySnapshot.forEach((docElement: any) => {
        this.scientists.push(docElement.data());
      });
    } catch (error) {
      console.error(error);
    }
  }

  // UPDATE
  async onSelect(event: any) { // type Event doesn't work despite https://angular.io/guide/event-binding-concepts
    try {
      this.querySnapshot = await getDocs(query(collection(this.firestore, 'Scientists'), where('name', '==', event.target.value))); // query the database
      this.querySnapshot.forEach((docElement: any) => { // itereate through the collection
        this.nameUpdate = docElement.data().name; // transfer to local variables
        this.bornUpdate = docElement.data().born;
        this.accomplishmentUpdate = docElement.data().accomplishment;
      });
    } catch (error) {
      console.error(error);
    }
  }

  // UPDATE
  async onUpdate() {
    try {
      this.querySnapshot = await getDocs(query(collection(this.firestore, 'Scientists'), where('name', '==', this.nameUpdate))); // find the document by the name property instead of the document identifier because we're using both autogenerated document identifiers and custom document identifiers
      this.querySnapshot.forEach((docElement: any) => { // iterate through the collection to find the document identifier for the selected document
        this.nameUpdate = docElement.id; // put the document identifier in a local variable
      });
      await updateDoc(doc(this.firestore, 'Scientists', this.nameUpdate), { // update the database
        born: this.bornUpdate,
        accomplishment: this.accomplishmentUpdate,
      });
      this.bornUpdate = null; // clear form field
      this.accomplishmentUpdate = null; // clear form field
    } catch (error) {
      console.error(error);
    }
  }

  // DELETE
  async onDelete() {
    try {
      this.querySnapshot = await getDocs(query(collection(this.firestore, 'Scientists'), where('name', '==', this.selection))); // get a collection of documents filtered by the query
      this.querySnapshot.forEach((docElement: any) => { // iterate through the collection
        deleteDoc(doc(this.firestore, 'Scientists', docElement.id)); // delete all documents that match the query
      });
    } catch (error) {
      console.error(error);
    }
  }

}
```
