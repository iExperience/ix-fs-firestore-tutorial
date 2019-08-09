# iXperience - Full Stack Day 16

Welcome to iXperience Full Stack 2019!

[[toc]]

## Angular Firestore

### Add FireStore as a provider

Import Angular Firestore in app.module.ts and add it as a provider

```ts

import { AngularFireModule } from '@angular/fire';
import { environment } from '../environments/environment';
import { AngularFirestore } from '@angular/fire/firestore';

@NgModule({
  declarations: [AppComponent],
  entryComponents: [],
  imports: [BrowserModule, IonicModule.forRoot(), AppRoutingModule, 
    AngularFireModule.initializeApp(environment.firebaseConfig) ],
  providers: [
    StatusBar,
    SplashScreen,
    AngularFirestore,
    { provide: RouteReuseStrategy, useClass: IonicRouteStrategy }
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}

```

### Create a Model

Create a model which will be used to define our table / document structure

```bash

ionic generate class models/message

```

In the model create parameters for an id, sentiment score, message and date created

```ts

export class Message {
  id: string;
  score: number;
  message: string;
  dateCreated: string;
}

```

### Create a Service

Create a service to perform database queries to firebase

```bash

ionic generate service services/message

```

In the service; import the AngularFirestore module and create CRUD (create, read, update and delete) methods to interface with firebase

```ts

import { Injectable } from '@angular/core';
import { AngularFirestore } from '@angular/fire/firestore';
import { Message } from '../models/message';

@Injectable({
  providedIn: 'root'
})
export class MessageService {

  constructor(private firestore: AngularFirestore) { 
  }

  getMessages() {
    return this.firestore.collection('messages').snapshotChanges();
  }

  createMessage(message: Message) {
    return this.firestore.collection('messages').add({...message});
  }

  updateMessage(message: Message): void {
    this.firestore.doc('messages/' + message.id).update(message);
  }

  deleteMessage(message: Message): void {
    this.firestore.doc('messages/' + message.id).delete();
  }

}


```

### Access Firebase using service in a component

Import the message service and implement local functions to get messages (includes mapping and ordering the response from firebase) and create messages.

```ts

import { Component, OnInit, } from '@angular/core';
import { MessageService } from '../services/message.service';
import { Message } from '../models/message';
    
@Component({
  selector: 'app-tab1',
  templateUrl: 'tab1.page.html',
  styleUrls: ['tab1.page.scss']
})
export class Tab1Page implements OnInit {

  messages: Message[];
  message: Message;

  constructor(private messageService: MessageService) {
    this.message = new Message();
    this.messages = [];
  }

  ngOnInit() {
  }

  ionViewWillEnter() {
    this.getMessages(); // runs get messages function every time the component is viewed
  }

  getMessages() {
    this.messageService.getMessages().subscribe(data => {
      this.messages = data.map(e => {
        return {
          id: e.payload.doc.id,
          ...e.payload.doc.data()
        } as Message;
      })
      this.messages.sort((message1, message2) => ((message1.dateCreated < message2.dateCreated) ? 1 : -1)); // order messages (optional)
    });
  }

  createMessage() {
    this.messageService.createMessage(this.message);
  }

}


```