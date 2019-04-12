# Step 11: Users

[//]: # (head-end)


Our chat app is pretty functional. We can pick a chat from the chats list and we can send messages. It's not hard to notice that one of the most important mechanisms is missing, which is relating a chat or a message to a specific user. Even though we can send messages, it's basically pointless unless someone else receives it. In this chapter we will create a new users collection with pre-defined documents and we will learn how to simulate authentication programmatically so we can test the new mechanism.

**Reshaping the back-end**

To implement this feature we need to rethink our back-end and reshape the way our GraphQL schema is structured. Right now we only have 2 entities: Chat and Message, which are connected like so:



![chat-message-orm](https://user-images.githubusercontent.com/7648874/55325929-0faa1b00-54b9-11e9-8868-7a8ed3edcda1.png)


We want to have a new User entity where each user will have Chats he participates in and Messages he owns. Therefore, our new GraphQL schema should look like something like this:



![chat-message-user-orm](https://user-images.githubusercontent.com/7648874/55325935-146ecf00-54b9-11e9-8c0f-bc3b63cbe676.png)

This change would require us to update the GraphQL type definitions and handlers, the DB models, and the codegen configuration file:

[{]: <helper> (diffStep 8.1 module="server")

#### [Step 8.1: Add User type](https://github.com/Urigo/WhatsApp-Clone-Server/commit/76ffcef)

##### Changed codegen.yml
```diff
@@ -10,6 +10,7 @@
 ┊10┊10┊      mappers:
 ┊11┊11┊        # import { Message } from '../db'
 ┊12┊12┊        # The root types of Message resolvers
+┊  ┊13┊        User: ../db#User
 ┊13┊14┊        Message: ../db#Message
 ┊14┊15┊        Chat: ../db#Chat
 ┊15┊16┊      scalars:
```

##### Changed db.ts
```diff
@@ -1,66 +1,106 @@
+┊   ┊  1┊export type User = {
+┊   ┊  2┊  id: string
+┊   ┊  3┊  name: string
+┊   ┊  4┊  picture: string
+┊   ┊  5┊}
+┊   ┊  6┊
 ┊  1┊  7┊export type Message = {
 ┊  2┊  8┊  id: string
 ┊  3┊  9┊  content: string
 ┊  4┊ 10┊  createdAt: Date
+┊   ┊ 11┊  sender: string
+┊   ┊ 12┊  recipient: string
 ┊  5┊ 13┊}
 ┊  6┊ 14┊
 ┊  7┊ 15┊export type Chat = {
 ┊  8┊ 16┊  id: string
-┊  9┊   ┊  name: string
-┊ 10┊   ┊  picture: string
 ┊ 11┊ 17┊  messages: string[]
+┊   ┊ 18┊  participants: string[]
 ┊ 12┊ 19┊}
 ┊ 13┊ 20┊
+┊   ┊ 21┊export const users: User[] = []
 ┊ 14┊ 22┊export const messages: Message[] = []
 ┊ 15┊ 23┊export const chats: Chat[] = []
 ┊ 16┊ 24┊
 ┊ 17┊ 25┊export const resetDb = () => {
+┊   ┊ 26┊  users.splice(0, Infinity, ...[
+┊   ┊ 27┊    {
+┊   ┊ 28┊      id: '1',
+┊   ┊ 29┊      name: 'Ray Edwards',
+┊   ┊ 30┊      picture: 'https://randomuser.me/api/portraits/thumb/lego/1.jpg',
+┊   ┊ 31┊    },
+┊   ┊ 32┊    {
+┊   ┊ 33┊      id: '2',
+┊   ┊ 34┊      name: 'Ethan Gonzalez',
+┊   ┊ 35┊      picture: 'https://randomuser.me/api/portraits/thumb/men/1.jpg',
+┊   ┊ 36┊    },
+┊   ┊ 37┊    {
+┊   ┊ 38┊      id: '3',
+┊   ┊ 39┊      name: 'Bryan Wallace',
+┊   ┊ 40┊      picture: 'https://randomuser.me/api/portraits/thumb/men/2.jpg',
+┊   ┊ 41┊    },
+┊   ┊ 42┊    {
+┊   ┊ 43┊      id: '4',
+┊   ┊ 44┊      name: 'Avery Stewart',
+┊   ┊ 45┊      picture: 'https://randomuser.me/api/portraits/thumb/women/1.jpg',
+┊   ┊ 46┊    },
+┊   ┊ 47┊    {
+┊   ┊ 48┊      id: '5',
+┊   ┊ 49┊      name: 'Katie Peterson',
+┊   ┊ 50┊      picture: 'https://randomuser.me/api/portraits/thumb/women/2.jpg',
+┊   ┊ 51┊    },
+┊   ┊ 52┊  ])
+┊   ┊ 53┊
 ┊ 18┊ 54┊  messages.splice(0, Infinity, ...[
 ┊ 19┊ 55┊    {
 ┊ 20┊ 56┊      id: '1',
 ┊ 21┊ 57┊      content: "You on your way?",
 ┊ 22┊ 58┊      createdAt: new Date(new Date('1-1-2019').getTime() - 60 * 1000 * 1000),
+┊   ┊ 59┊      sender: '1',
+┊   ┊ 60┊      recipient: '2',
 ┊ 23┊ 61┊    },
 ┊ 24┊ 62┊    {
 ┊ 25┊ 63┊      id: '2',
 ┊ 26┊ 64┊      content: "Hey, it's me",
 ┊ 27┊ 65┊      createdAt: new Date(new Date('1-1-2019').getTime() - 2 * 60 * 1000 * 1000),
+┊   ┊ 66┊      sender: '1',
+┊   ┊ 67┊      recipient: '3',
 ┊ 28┊ 68┊    },
 ┊ 29┊ 69┊    {
 ┊ 30┊ 70┊      id: '3',
 ┊ 31┊ 71┊      content: "I should buy a boat",
 ┊ 32┊ 72┊      createdAt: new Date(new Date('1-1-2019').getTime() - 24 * 60 * 1000 * 1000),
+┊   ┊ 73┊      sender: '1',
+┊   ┊ 74┊      recipient: '4',
 ┊ 33┊ 75┊    },
 ┊ 34┊ 76┊    {
 ┊ 35┊ 77┊      id: '4',
 ┊ 36┊ 78┊      content: "This is wicked good ice cream.",
 ┊ 37┊ 79┊      createdAt: new Date(new Date('1-1-2019').getTime() - 14 * 24 * 60 * 1000 * 1000),
+┊   ┊ 80┊      sender: '1',
+┊   ┊ 81┊      recipient: '5',
 ┊ 38┊ 82┊    },
 ┊ 39┊ 83┊  ])
 ┊ 40┊ 84┊
 ┊ 41┊ 85┊  chats.splice(0, Infinity, ...[
 ┊ 42┊ 86┊    {
 ┊ 43┊ 87┊      id: '1',
-┊ 44┊   ┊      name: 'Ethan Gonzalez',
-┊ 45┊   ┊      picture: 'https://randomuser.me/api/portraits/thumb/men/1.jpg',
+┊   ┊ 88┊      participants: ['1', '2'],
 ┊ 46┊ 89┊      messages: ['1'],
 ┊ 47┊ 90┊    },
 ┊ 48┊ 91┊    {
 ┊ 49┊ 92┊      id: '2',
-┊ 50┊   ┊      name: 'Bryan Wallace',
-┊ 51┊   ┊      picture: 'https://randomuser.me/api/portraits/thumb/men/2.jpg',
+┊   ┊ 93┊      participants: ['1', '3'],
 ┊ 52┊ 94┊      messages: ['2'],
 ┊ 53┊ 95┊    },
 ┊ 54┊ 96┊    {
 ┊ 55┊ 97┊      id: '3',
-┊ 56┊   ┊      name: 'Avery Stewart',
-┊ 57┊   ┊      picture: 'https://randomuser.me/api/portraits/thumb/women/1.jpg',
+┊   ┊ 98┊      participants: ['1', '4'],
 ┊ 58┊ 99┊      messages: ['3'],
 ┊ 59┊100┊    },
 ┊ 60┊101┊    {
 ┊ 61┊102┊      id: '4',
-┊ 62┊   ┊      name: 'Katie Peterson',
-┊ 63┊   ┊      picture: 'https://randomuser.me/api/portraits/thumb/women/2.jpg',
+┊   ┊103┊      participants: ['1', '5'],
 ┊ 64┊104┊      messages: ['4'],
 ┊ 65┊105┊    },
 ┊ 66┊106┊  ])
```

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -1,5 +1,5 @@
 ┊1┊1┊import { GraphQLDateTime } from 'graphql-iso-date'
-┊2┊ ┊import { Message, chats, messages } from '../db'
+┊ ┊2┊import { User, Message, chats, messages, users } from '../db'
 ┊3┊3┊import { Resolvers } from '../types/graphql'
 ┊4┊4┊
 ┊5┊5┊const resolvers: Resolvers = {
```
```diff
@@ -9,9 +9,27 @@
 ┊ 9┊ 9┊    chat(message) {
 ┊10┊10┊      return chats.find(c => c.messages.some(m => m === message.id)) || null
 ┊11┊11┊    },
+┊  ┊12┊
+┊  ┊13┊    sender(message) {
+┊  ┊14┊      return users.find(u => u.id === message.sender) || null
+┊  ┊15┊    },
+┊  ┊16┊
+┊  ┊17┊    recipient(message) {
+┊  ┊18┊      return users.find(u => u.id === message.recipient) || null
+┊  ┊19┊    },
 ┊12┊20┊  },
 ┊13┊21┊
 ┊14┊22┊  Chat: {
+┊  ┊23┊    name() {
+┊  ┊24┊      // TODO: Resolve in relation to current user
+┊  ┊25┊      return null
+┊  ┊26┊    },
+┊  ┊27┊
+┊  ┊28┊    picture() {
+┊  ┊29┊      // TODO: Resolve in relation to current user
+┊  ┊30┊      return null
+┊  ┊31┊    },
+┊  ┊32┊
 ┊15┊33┊    messages(chat) {
 ┊16┊34┊      return messages.filter(m => chat.messages.includes(m.id))
 ┊17┊35┊    },
```
```diff
@@ -21,6 +39,10 @@
 ┊21┊39┊
 ┊22┊40┊      return messages.find(m => m.id === lastMessage) || null
 ┊23┊41┊    },
+┊  ┊42┊
+┊  ┊43┊    participants(chat) {
+┊  ┊44┊      return chat.participants.map(p => users.find(u => u.id === p)).filter(Boolean) as User[]
+┊  ┊45┊    },
 ┊24┊46┊  },
 ┊25┊47┊
 ┊26┊48┊  Query: {
```
```diff
@@ -45,6 +67,8 @@
 ┊45┊67┊      const message: Message = {
 ┊46┊68┊        id: messageId,
 ┊47┊69┊        createdAt: new Date(),
+┊  ┊70┊        sender: '', // TODO: Fill-in
+┊  ┊71┊        recipient: '', // TODO: Fill-in
 ┊48┊72┊        content,
 ┊49┊73┊      }
 ┊50┊74┊
```

##### Changed schema&#x2F;typeDefs.graphql
```diff
@@ -1,18 +1,28 @@
 ┊ 1┊ 1┊scalar Date
 ┊ 2┊ 2┊
+┊  ┊ 3┊type User {
+┊  ┊ 4┊  id: ID!
+┊  ┊ 5┊  name: String!
+┊  ┊ 6┊  picture: String
+┊  ┊ 7┊}
+┊  ┊ 8┊
 ┊ 3┊ 9┊type Message {
 ┊ 4┊10┊  id: ID!
 ┊ 5┊11┊  content: String!
 ┊ 6┊12┊  createdAt: Date!
 ┊ 7┊13┊  chat: Chat
+┊  ┊14┊  sender: User
+┊  ┊15┊  recipient: User
+┊  ┊16┊  isMine: Boolean!
 ┊ 8┊17┊}
 ┊ 9┊18┊
 ┊10┊19┊type Chat {
 ┊11┊20┊  id: ID!
-┊12┊  ┊  name: String!
+┊  ┊21┊  name: String
 ┊13┊22┊  picture: String
 ┊14┊23┊  lastMessage: Message
 ┊15┊24┊  messages: [Message!]!
+┊  ┊25┊  participants: [User!]!
 ┊16┊26┊}
 ┊17┊27┊
 ┊18┊28┊type Query {
```

[}]: #

Even though we made these changes, the app remained the same. That's because the Query type haven't changed at all, and we still serve the same data as before. What we need to do is to edit the Query resolvers to serve data based on the user that is currently logged-in to the app in the current session. Before we go all in with a robust authentication system, it would be smarter to simulate it, so we can test our app and see that everything works as intended.

For now, let's assume that we're logged in with user of ID 1 - Ray Edwards. Codewise, this would mean that we will need to have the current user defined on the resolver context. In the main file, let's add the `currentUser` field to the context using a simple `find()` method from our `users` collection:

[{]: <helper> (diffStep 8.2 files="index.ts" module="server")

#### [Step 8.2: Resolve queries in relation to current user](https://github.com/Urigo/WhatsApp-Clone-Server/commit/cc56fb3)

##### Changed index.ts
```diff
@@ -3,6 +3,7 @@
 ┊3┊3┊import cors from 'cors'
 ┊4┊4┊import express from 'express'
 ┊5┊5┊import http from 'http'
+┊ ┊6┊import { users } from './db'
 ┊6┊7┊import schema from './schema'
 ┊7┊8┊
 ┊8┊9┊const app = express()
```
```diff
@@ -17,7 +18,10 @@
 ┊17┊18┊const pubsub = new PubSub()
 ┊18┊19┊const server = new ApolloServer({
 ┊19┊20┊  schema,
-┊20┊  ┊  context: () => ({ pubsub }),
+┊  ┊21┊  context: () => ({
+┊  ┊22┊    currentUser: users.find(u => u.id === '1'),
+┊  ┊23┊    pubsub,
+┊  ┊24┊  }),
 ┊21┊25┊})
 ┊22┊26┊
 ┊23┊27┊server.applyMiddleware({
```

[}]: #

And we will update the context type:

[{]: <helper> (diffStep 8.2 files="context" module="server")

#### [Step 8.2: Resolve queries in relation to current user](https://github.com/Urigo/WhatsApp-Clone-Server/commit/cc56fb3)

##### Changed context.ts
```diff
@@ -1,5 +1,7 @@
 ┊1┊1┊import { PubSub } from 'apollo-server-express'
+┊ ┊2┊import { User } from './db'
 ┊2┊3┊
 ┊3┊4┊export type MyContext = {
-┊4┊ ┊  pubsub: PubSub
+┊ ┊5┊  pubsub: PubSub,
+┊ ┊6┊  currentUser: User,
 ┊5┊7┊}
```

[}]: #

Now we will update the resolvers to fetch data relatively to the current user logged in. If there's no user logged in, the resolvers should return `null`, as the client is not authorized to view the data he requested:

[{]: <helper> (diffStep 8.2 files="schema, tests" module="server")

#### [Step 8.2: Resolve queries in relation to current user](https://github.com/Urigo/WhatsApp-Clone-Server/commit/cc56fb3)

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -17,17 +17,35 @@
 ┊17┊17┊    recipient(message) {
 ┊18┊18┊      return users.find(u => u.id === message.recipient) || null
 ┊19┊19┊    },
+┊  ┊20┊
+┊  ┊21┊    isMine(message, args, { currentUser }) {
+┊  ┊22┊      return message.sender === currentUser.id
+┊  ┊23┊    },
 ┊20┊24┊  },
 ┊21┊25┊
 ┊22┊26┊  Chat: {
-┊23┊  ┊    name() {
-┊24┊  ┊      // TODO: Resolve in relation to current user
-┊25┊  ┊      return null
+┊  ┊27┊    name(chat, args, { currentUser }) {
+┊  ┊28┊      if (!currentUser) return null
+┊  ┊29┊
+┊  ┊30┊      const participantId = chat.participants.find(p => p !== currentUser.id)
+┊  ┊31┊
+┊  ┊32┊      if (!participantId) return null
+┊  ┊33┊
+┊  ┊34┊      const participant = users.find(u => u.id === participantId)
+┊  ┊35┊
+┊  ┊36┊      return participant ? participant.name : null
 ┊26┊37┊    },
 ┊27┊38┊
-┊28┊  ┊    picture() {
-┊29┊  ┊      // TODO: Resolve in relation to current user
-┊30┊  ┊      return null
+┊  ┊39┊    picture(chat, args, { currentUser }) {
+┊  ┊40┊      if (!currentUser) return null
+┊  ┊41┊
+┊  ┊42┊      const participantId = chat.participants.find(p => p !== currentUser.id)
+┊  ┊43┊
+┊  ┊44┊      if (!participantId) return null
+┊  ┊45┊
+┊  ┊46┊      const participant = users.find(u => u.id === participantId)
+┊  ┊47┊
+┊  ┊48┊      return participant ? participant.picture : null
 ┊31┊49┊    },
 ┊32┊50┊
 ┊33┊51┊    messages(chat) {
```
```diff
@@ -46,29 +64,42 @@
 ┊ 46┊ 64┊  },
 ┊ 47┊ 65┊
 ┊ 48┊ 66┊  Query: {
-┊ 49┊   ┊    chats() {
-┊ 50┊   ┊      return chats
+┊   ┊ 67┊    chats(root, args, { currentUser }) {
+┊   ┊ 68┊      if (!currentUser) return []
+┊   ┊ 69┊
+┊   ┊ 70┊      return chats.filter(c => c.participants.includes(currentUser.id))
 ┊ 51┊ 71┊    },
 ┊ 52┊ 72┊
-┊ 53┊   ┊    chat(root, { chatId }) {
-┊ 54┊   ┊      return chats.find(c => c.id === chatId) || null
+┊   ┊ 73┊    chat(root, { chatId }, { currentUser }) {
+┊   ┊ 74┊      if (!currentUser) return null
+┊   ┊ 75┊
+┊   ┊ 76┊      const chat = chats.find(c => c.id === chatId)
+┊   ┊ 77┊
+┊   ┊ 78┊      if (!chat) return null
+┊   ┊ 79┊
+┊   ┊ 80┊      return chat.participants.includes(currentUser.id) ? chat : null
 ┊ 55┊ 81┊    },
 ┊ 56┊ 82┊  },
 ┊ 57┊ 83┊
 ┊ 58┊ 84┊  Mutation: {
-┊ 59┊   ┊    addMessage(root, { chatId, content }, { pubsub }) {
+┊   ┊ 85┊    addMessage(root, { chatId, content }, { currentUser, pubsub }) {
+┊   ┊ 86┊      if (!currentUser) return null
+┊   ┊ 87┊
 ┊ 60┊ 88┊      const chatIndex = chats.findIndex(c => c.id === chatId)
 ┊ 61┊ 89┊
 ┊ 62┊ 90┊      if (chatIndex === -1) return null
 ┊ 63┊ 91┊
 ┊ 64┊ 92┊      const chat = chats[chatIndex]
+┊   ┊ 93┊
+┊   ┊ 94┊      if (!chat.participants.includes(currentUser.id)) return null
+┊   ┊ 95┊
 ┊ 65┊ 96┊      const recentMessage = messages[messages.length - 1]
 ┊ 66┊ 97┊      const messageId = String(Number(recentMessage.id) + 1)
 ┊ 67┊ 98┊      const message: Message = {
 ┊ 68┊ 99┊        id: messageId,
 ┊ 69┊100┊        createdAt: new Date(),
-┊ 70┊   ┊        sender: '', // TODO: Fill-in
-┊ 71┊   ┊        recipient: '', // TODO: Fill-in
+┊   ┊101┊        sender: currentUser.id,
+┊   ┊102┊        recipient: chat.participants.find(p => p !== currentUser.id) as string,
 ┊ 72┊103┊        content,
 ┊ 73┊104┊      }
 ┊ 74┊105┊
```

##### Changed tests&#x2F;mutations&#x2F;addMessage.test.ts
```diff
@@ -1,7 +1,7 @@
 ┊1┊1┊import { createTestClient } from 'apollo-server-testing'
 ┊2┊2┊import { ApolloServer, PubSub, gql } from 'apollo-server-express'
 ┊3┊3┊import schema from '../../schema'
-┊4┊ ┊import { resetDb } from '../../db'
+┊ ┊4┊import { resetDb, users } from '../../db'
 ┊5┊5┊
 ┊6┊6┊describe('Mutation.addMessage', () => {
 ┊7┊7┊  beforeEach(resetDb)
```
```diff
@@ -9,7 +9,10 @@
 ┊ 9┊ 9┊  it('should add message to specified chat', async () => {
 ┊10┊10┊    const server = new ApolloServer({
 ┊11┊11┊      schema,
-┊12┊  ┊      context: () => ({ pubsub: new PubSub() }),
+┊  ┊12┊      context: () => ({
+┊  ┊13┊        pubsub: new PubSub(),
+┊  ┊14┊        currentUser: users[0],
+┊  ┊15┊      }),
 ┊13┊16┊    })
 ┊14┊17┊
 ┊15┊18┊    const { query, mutate } = createTestClient(server)
```

##### Changed tests&#x2F;queries&#x2F;getChat.test.ts
```diff
@@ -1,10 +1,16 @@
 ┊ 1┊ 1┊import { createTestClient } from 'apollo-server-testing'
 ┊ 2┊ 2┊import { ApolloServer, gql } from 'apollo-server-express'
 ┊ 3┊ 3┊import schema from '../../schema'
+┊  ┊ 4┊import { users } from '../../db'
 ┊ 4┊ 5┊
 ┊ 5┊ 6┊describe('Query.chat', () => {
 ┊ 6┊ 7┊  it('should fetch specified chat', async () => {
-┊ 7┊  ┊    const server = new ApolloServer({ schema })
+┊  ┊ 8┊    const server = new ApolloServer({
+┊  ┊ 9┊      schema,
+┊  ┊10┊      context: () => ({
+┊  ┊11┊        currentUser: users[0],
+┊  ┊12┊      }),
+┊  ┊13┊    })
 ┊ 8┊14┊
 ┊ 9┊15┊    const { query } = createTestClient(server)
 ┊10┊16┊
```

##### Changed tests&#x2F;queries&#x2F;getChats.test.ts
```diff
@@ -1,10 +1,16 @@
 ┊ 1┊ 1┊import { createTestClient } from 'apollo-server-testing'
 ┊ 2┊ 2┊import { ApolloServer, gql } from 'apollo-server-express'
 ┊ 3┊ 3┊import schema from '../../schema'
+┊  ┊ 4┊import { users } from '../../db'
 ┊ 4┊ 5┊
 ┊ 5┊ 6┊describe('Query.chats', () => {
 ┊ 6┊ 7┊  it('should fetch all chats', async () => {
-┊ 7┊  ┊    const server = new ApolloServer({ schema })
+┊  ┊ 8┊    const server = new ApolloServer({
+┊  ┊ 9┊      schema,
+┊  ┊10┊      context: () => ({
+┊  ┊11┊        currentUser: users[0],
+┊  ┊12┊      }),
+┊  ┊13┊    })
 ┊ 8┊14┊
 ┊ 9┊15┊    const { query } = createTestClient(server)
```

[}]: #

Now if we will get back to the app and refresh the page, we should see a new chats list which is only relevant to Ray Edwards. Earlier in this chapter, we've defined a new `isMine` field on the `Message` type. This field is useful because now we can differentiate between messages that are mine and messages that belong to the recipient. We can use that information to distinct between messages in our UI.

Let's first download a new image that will help us achieve the new style and save it under the [`src/public/assets/message-yours.png`](https://github.com/Urigo/WhatsApp-Clone-Client-React/blob/cordova/public/assets/message-other.png?raw=true) path. Then let's implement the new style:

[{]: <helper> (diffStep 11.1 files="src/components" module="client")

#### [Step 11.1: Distinguish messages](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/7e36442)

##### Changed src&#x2F;components&#x2F;ChatRoomScreen&#x2F;MessagesList.tsx
```diff
@@ -2,7 +2,7 @@
 ┊2┊2┊import * as React from 'react'
 ┊3┊3┊import { useEffect, useRef } from 'react'
 ┊4┊4┊import * as ReactDOM from 'react-dom'
-┊5┊ ┊import styled from 'styled-components'
+┊ ┊5┊import styled, { css } from 'styled-components'
 ┊6┊6┊
 ┊7┊7┊const Container = styled.div`
 ┊8┊8┊  display: block;
```
```diff
@@ -12,8 +12,6 @@
 ┊12┊12┊`
 ┊13┊13┊
 ┊14┊14┊const MessageItem = styled.div `
-┊15┊  ┊  float: right;
-┊16┊  ┊  background-color: #dcf8c6;
 ┊17┊15┊  display: inline-block;
 ┊18┊16┊  position: relative;
 ┊19┊17┊  max-width: 100%;
```
```diff
@@ -30,17 +28,33 @@
 ┊30┊28┊  }
 ┊31┊29┊
 ┊32┊30┊  &::before {
-┊33┊  ┊    background-image: url(/assets/message-mine.png);
 ┊34┊31┊    content: '';
 ┊35┊32┊    position: absolute;
 ┊36┊33┊    bottom: 3px;
 ┊37┊34┊    width: 12px;
 ┊38┊35┊    height: 19px;
-┊39┊  ┊    right: -11px;
 ┊40┊36┊    background-position: 50% 50%;
 ┊41┊37┊    background-repeat: no-repeat;
 ┊42┊38┊    background-size: contain;
 ┊43┊39┊  }
+┊  ┊40┊
+┊  ┊41┊  ${props => props.isMine ? css `
+┊  ┊42┊    float: right;
+┊  ┊43┊    background-color: #dcf8c6;
+┊  ┊44┊
+┊  ┊45┊    &::before {
+┊  ┊46┊      right: -11px;
+┊  ┊47┊      background-image: url(/assets/message-mine.png);
+┊  ┊48┊    }
+┊  ┊49┊  ` : css `
+┊  ┊50┊    float: left;
+┊  ┊51┊    background-color: #fff;
+┊  ┊52┊
+┊  ┊53┊    &::before {
+┊  ┊54┊      left: -11px;
+┊  ┊55┊      background-image: url(/assets/message-other.png);
+┊  ┊56┊    }
+┊  ┊57┊  `}
 ┊44┊58┊`
 ┊45┊59┊
 ┊46┊60┊const Contents = styled.div `
```
```diff
@@ -74,7 +88,11 @@
 ┊74┊88┊  return (
 ┊75┊89┊    <Container ref={selfRef}>
 ┊76┊90┊      {messages.map((message) => (
-┊77┊  ┊        <MessageItem data-testid="message-item" key={message.id}>
+┊  ┊91┊        <MessageItem
+┊  ┊92┊          data-testid="message-item"
+┊  ┊93┊          isMine={message.isMine}
+┊  ┊94┊          key={message.id}
+┊  ┊95┊        >
 ┊78┊96┊          <Contents data-testid="message-content">{message.content}</Contents>
 ┊79┊97┊          <Timestamp data-testid="message-date">{moment(message.createdAt).format('HH:mm')}</Timestamp>
 ┊80┊98┊        </MessageItem>
```

##### Changed src&#x2F;components&#x2F;ChatRoomScreen&#x2F;index.tsx
```diff
@@ -54,6 +54,7 @@
 ┊54┊54┊          __typename: 'Message',
 ┊55┊55┊          id: Math.random().toString(36).substr(2, 9),
 ┊56┊56┊          createdAt: new Date(),
+┊  ┊57┊          isMine: true,
 ┊57┊58┊          chat: {
 ┊58┊59┊            __typename: 'Chat',
 ┊59┊60┊            id: chatId,
```

##### Changed src&#x2F;components&#x2F;ChatsListScreen&#x2F;ChatsList.test.tsx
```diff
@@ -30,6 +30,7 @@
 ┊30┊30┊                  id: 1,
 ┊31┊31┊                  content: 'Hello',
 ┊32┊32┊                  createdAt: new Date(0),
+┊  ┊33┊                  isMine: true,
 ┊33┊34┊                  chat: {
 ┊34┊35┊                    __typename: 'Chat',
 ┊35┊36┊                    id: 1,
```
```diff
@@ -75,6 +76,7 @@
 ┊75┊76┊                  id: 1,
 ┊76┊77┊                  content: 'Hello',
 ┊77┊78┊                  createdAt: new Date(0),
+┊  ┊79┊                  isMine: true,
 ┊78┊80┊                  chat: {
 ┊79┊81┊                    __typename: 'Chat',
 ┊80┊82┊                    id: 1,
```

[}]: #

This is how the updated `ChatRoomScreen` should look like:



![chat-room-screen](https://user-images.githubusercontent.com/7648874/55326701-face8700-54ba-11e9-877e-0b7dd71a1b68.png)



We can use a temporary solution to log-in and alternate between different users. This would be a good way to test data authorization without implementing an authentication mechanism. One way to know which user is logged in is via [cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies).

Cookies are just text files which are stored locally on your computer and they contain key-value data maps. Cookies will be sent automatically by the browser with every HTTP request under the `Cookie` header. The header can be parsed and read by the server and this way inform it about the state of the client. Cookie values can also be set by the server by sending back a response which contain a `Set-Cookie` header. The browser will automatically write these cookies because of its specification and how it works.

This is how you can set cookies on the client:

```js
document.cookie = "yummy_cookie=choco"
document.cookie = "tasty_cookie=strawberry"
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
```

And this is how further requests would look like:

```
GET /sample_page.html HTTP/2.0
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

Using this method we can set the current user's ID. Open your browser's dev-console, and type the following:

```js
// Ray Edwards
document.cookie = 'currentUserId=1'
```

To be able to send cookies with Apollo Client, we need to set the [`credentials`](https://www.apollographql.com/docs/react/recipes/authentication#cookie) option to "include" when creating the HTTP link:

[{]: <helper> (diffStep 11.2 module="client")

#### [Step 11.2: Support credentials](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/e042471)

##### Changed src&#x2F;client.ts
```diff
@@ -10,6 +10,7 @@
 ┊10┊10┊
 ┊11┊11┊const httpLink = new HttpLink({
 ┊12┊12┊  uri: httpUri,
+┊  ┊13┊  credentials: 'include',
 ┊13┊14┊})
 ┊14┊15┊
 ┊15┊16┊const wsLink = new WebSocketLink({
```

[}]: #

This will set the [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) header to “include” with each HTTP request which is necessary when using the POST method. In correlation to that, we would need to configure the server to be able to receive and set cookies. This can be done via CORS options like so:

[{]: <helper> (diffStep 8.4 files="index.ts" module="server")

#### [Step 8.4: Support credentials](https://github.com/Urigo/WhatsApp-Clone-Server/commit/d65c2f3)

##### Changed index.ts
```diff
@@ -8,7 +8,8 @@
 ┊ 8┊ 8┊
 ┊ 9┊ 9┊const app = express()
 ┊10┊10┊
-┊11┊  ┊app.use(cors())
+┊  ┊11┊const origin = process.env.ORIGIN || 'http://localhost:3000'
+┊  ┊12┊app.use(cors({ credentials: true, origin }))
 ┊12┊13┊app.use(bodyParser.json())
 ┊13┊14┊
 ┊14┊15┊app.get('/_ping', (req, res) => {
```
```diff
@@ -27,6 +28,7 @@
 ┊27┊28┊server.applyMiddleware({
 ┊28┊29┊  app,
 ┊29┊30┊  path: '/graphql',
+┊  ┊31┊  cors: { credentials: true, origin },
 ┊30┊32┊})
 ┊31┊33┊
 ┊32┊34┊const httpServer = http.createServer(app)
```

[}]: #

So how exactly does one retrieve the values of the cookies? Like mentioned earlier, each and every request will have them set on the `cookie` header, so one way would be by reading the header directly, but a more convenient way would be using an Express middleware called [`cookie-parser`](https://www.npmjs.com/package/cookie-parser):

  $ npm install cookie-parser

[{]: <helper> (diffStep 8.5 files="index.ts" module="server")

#### [Step 8.5: Use cookie parser](https://github.com/Urigo/WhatsApp-Clone-Server/commit/2cd3964)

##### Changed index.ts
```diff
@@ -1,6 +1,7 @@
 ┊1┊1┊import { ApolloServer, gql, PubSub } from 'apollo-server-express'
 ┊2┊2┊import bodyParser from 'body-parser'
 ┊3┊3┊import cors from 'cors'
+┊ ┊4┊import cookieParser from 'cookie-parser'
 ┊4┊5┊import express from 'express'
 ┊5┊6┊import http from 'http'
 ┊6┊7┊import { users } from './db'
```
```diff
@@ -11,6 +12,7 @@
 ┊11┊12┊const origin = process.env.ORIGIN || 'http://localhost:3000'
 ┊12┊13┊app.use(cors({ credentials: true, origin }))
 ┊13┊14┊app.use(bodyParser.json())
+┊  ┊15┊app.use(cookieParser())
 ┊14┊16┊
 ┊15┊17┊app.get('/_ping', (req, res) => {
 ┊16┊18┊  res.send('pong')
```

[}]: #

`cookie-parser` will read the `Cookie` header, it will parse it into a JSON and will define it on `req.cookies`. Since we’re using Apollo-Server with Express, the `req` object should be accessible as the first argument in the `context` function. This means that we can use the `currentUserId` from the cookies to fetch the current user from our users collection and define it on the context object:

[{]: <helper> (diffStep 8.6 module="server")

#### [Step 8.6: Define current user based on cookies](https://github.com/Urigo/WhatsApp-Clone-Server/commit/bdbae75)

##### Changed index.ts
```diff
@@ -21,8 +21,8 @@
 ┊21┊21┊const pubsub = new PubSub()
 ┊22┊22┊const server = new ApolloServer({
 ┊23┊23┊  schema,
-┊24┊  ┊  context: () => ({
-┊25┊  ┊    currentUser: users.find(u => u.id === '1'),
+┊  ┊24┊  context: ({ req }) => ({
+┊  ┊25┊    currentUser: users.find(u => u.id === req.cookies.currentUserId),
 ┊26┊26┊    pubsub,
 ┊27┊27┊  }),
 ┊28┊28┊})
```

[}]: #

Now you can go ahead and change the value of the `currentUserId` cookie and see how it affects the view anytime you refresh the page. Needless to say that this is not the most convenient way to switch between users, so we’re gonna implement a dedicated screen that will set the cookies for us.

All the auth related logic should go into a dedicated service since it can serve us vastly across the application, not just for a single component. Thus we will create a new service called `auth.service`, which will contain 3 basic functions for now: `signIn()`, `signOut()` and `isSignedIn():

[{]: <helper> (diffStep 11.3 module="client")

#### [Step 11.3: Add basic auth.service](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/adbd108)

##### Added src&#x2F;services&#x2F;auth.service.tsx
```diff
@@ -0,0 +1,21 @@
+┊  ┊ 1┊import client from '../client'
+┊  ┊ 2┊
+┊  ┊ 3┊export const signIn = (currentUserId) => {
+┊  ┊ 4┊  document.cookie = `currentUserId=${currentUserId}`
+┊  ┊ 5┊
+┊  ┊ 6┊  // This will become async in the near future
+┊  ┊ 7┊  return Promise.resolve()
+┊  ┊ 8┊}
+┊  ┊ 9┊
+┊  ┊10┊export const signOut = () => {
+┊  ┊11┊  // "expires" represents the lifespan of a cookie. Beyond that date the cookie will
+┊  ┊12┊  // be deleted by the browser. "expires" cannot be viewed from "document.cookie"
+┊  ┊13┊  document.cookie = `currentUserId=;expires=${new Date(0)}`
+┊  ┊14┊
+┊  ┊15┊  // Clear cache
+┊  ┊16┊  return client.clearStore()
+┊  ┊17┊}
+┊  ┊18┊
+┊  ┊19┊export const isSignedIn = () => {
+┊  ┊20┊  return /currentUserId=.+(;|$)/.test(document.cookie)
+┊  ┊21┊}
```

[}]: #

Now we will implement the `AuthScreen`. For now this screen should be fairly simple. It should contain a single `TextField` to specify the current user ID, and a `sign-in` button that will call the `signIn()` method with the specified ID. Once it does so, we will be proceeded to the `ChatsListScreen`. First we will download and save the following assets:

- [`src/public/assets/whatsapp-icon.ping`](https://github.com/Urigo/WhatsApp-Clone-Client-React/raw/wip/cookie-auth/public/assets/whatsapp-icon.png)

[{]: <helper> (diffStep 11.4 files="components" module="client")

#### [Step 11.4: Add AuthScreen](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/647bb65)

##### Added src&#x2F;components&#x2F;AuthScreen&#x2F;index.tsx
```diff
@@ -0,0 +1,162 @@
+┊   ┊  1┊import MaterialButton from '@material-ui/core/Button'
+┊   ┊  2┊import MaterialTextField from '@material-ui/core/TextField'
+┊   ┊  3┊import * as React from 'react'
+┊   ┊  4┊import { useCallback, useMemo, useState } from 'react'
+┊   ┊  5┊import styled from 'styled-components'
+┊   ┊  6┊import { signIn } from '../../services/auth.service'
+┊   ┊  7┊
+┊   ┊  8┊const Container = styled.div `
+┊   ┊  9┊  height: 100%;
+┊   ┊ 10┊  background: radial-gradient(rgb(34, 65, 67), rgb(17, 48, 50)),
+┊   ┊ 11┊    url(/assets/chat-background.jpg) no-repeat;
+┊   ┊ 12┊  background-size: cover;
+┊   ┊ 13┊  background-blend-mode: multiply;
+┊   ┊ 14┊  color: white;
+┊   ┊ 15┊`
+┊   ┊ 16┊
+┊   ┊ 17┊const Intro = styled.div `
+┊   ┊ 18┊  height: 265px;
+┊   ┊ 19┊`
+┊   ┊ 20┊
+┊   ┊ 21┊const Icon = styled.img `
+┊   ┊ 22┊  width: 125px;
+┊   ┊ 23┊  height: auto;
+┊   ┊ 24┊  margin-left: auto;
+┊   ┊ 25┊  margin-right: auto;
+┊   ┊ 26┊  padding-top: 70px;
+┊   ┊ 27┊  display: block;
+┊   ┊ 28┊`
+┊   ┊ 29┊
+┊   ┊ 30┊const Title = styled.h2 `
+┊   ┊ 31┊  width: 100%;
+┊   ┊ 32┊  text-align: center;
+┊   ┊ 33┊  color: white;
+┊   ┊ 34┊`
+┊   ┊ 35┊
+┊   ┊ 36┊const Alternative = styled.div `
+┊   ┊ 37┊  position: fixed;
+┊   ┊ 38┊  bottom: 10px;
+┊   ┊ 39┊  left: 10px;
+┊   ┊ 40┊
+┊   ┊ 41┊  a {
+┊   ┊ 42┊    color: var(--secondary-bg);
+┊   ┊ 43┊  }
+┊   ┊ 44┊`
+┊   ┊ 45┊
+┊   ┊ 46┊const SignInForm = styled.div `
+┊   ┊ 47┊  height: calc(100% - 265px);
+┊   ┊ 48┊`
+┊   ┊ 49┊
+┊   ┊ 50┊const ActualForm = styled.form `
+┊   ┊ 51┊  padding: 20px;
+┊   ┊ 52┊`
+┊   ┊ 53┊
+┊   ┊ 54┊const Section = styled.div `
+┊   ┊ 55┊  width: 100%;
+┊   ┊ 56┊  padding-bottom: 35px;
+┊   ┊ 57┊`
+┊   ┊ 58┊
+┊   ┊ 59┊const Legend = styled.legend `
+┊   ┊ 60┊  font-weight: bold;
+┊   ┊ 61┊  color: white;
+┊   ┊ 62┊`
+┊   ┊ 63┊
+┊   ┊ 64┊const Label = styled.label `
+┊   ┊ 65┊  color: white !important;
+┊   ┊ 66┊`
+┊   ┊ 67┊
+┊   ┊ 68┊const Input = styled.input `
+┊   ┊ 69┊  color: white;
+┊   ┊ 70┊
+┊   ┊ 71┊  &::placeholder {
+┊   ┊ 72┊    color: var(--primary-bg);
+┊   ┊ 73┊  }
+┊   ┊ 74┊`
+┊   ┊ 75┊
+┊   ┊ 76┊const TextField = styled(MaterialTextField) `
+┊   ┊ 77┊  width: 100%;
+┊   ┊ 78┊  position: relative;
+┊   ┊ 79┊
+┊   ┊ 80┊  > div::before {
+┊   ┊ 81┊    border-color: white !important;
+┊   ┊ 82┊  }
+┊   ┊ 83┊
+┊   ┊ 84┊  input {
+┊   ┊ 85┊    color: white !important;
+┊   ┊ 86┊
+┊   ┊ 87┊    &::placeholder {
+┊   ┊ 88┊      color: var(--primary-bg) !important;
+┊   ┊ 89┊    }
+┊   ┊ 90┊  }
+┊   ┊ 91┊
+┊   ┊ 92┊  label {
+┊   ┊ 93┊    color: white !important;
+┊   ┊ 94┊  }
+┊   ┊ 95┊`
+┊   ┊ 96┊
+┊   ┊ 97┊const Button = styled(MaterialButton) `
+┊   ┊ 98┊  width: 100px;
+┊   ┊ 99┊  display: block !important;
+┊   ┊100┊  margin: auto !important;
+┊   ┊101┊  background-color: var(--secondary-bg) !important;
+┊   ┊102┊
+┊   ┊103┊  &[disabled] {
+┊   ┊104┊    color: #38a81c;
+┊   ┊105┊  }
+┊   ┊106┊
+┊   ┊107┊  &:not([disabled]) {
+┊   ┊108┊    color: white;
+┊   ┊109┊  }
+┊   ┊110┊`
+┊   ┊111┊
+┊   ┊112┊export default ({ history }) => {
+┊   ┊113┊  const [userId, setUserId] = useState('')
+┊   ┊114┊
+┊   ┊115┊  const onUserIdChange = useCallback(({ target }) => {
+┊   ┊116┊    setUserId(target.value)
+┊   ┊117┊  }, [true])
+┊   ┊118┊
+┊   ┊119┊  const maySignIn = useCallback(() => {
+┊   ┊120┊    return !!userId
+┊   ┊121┊  }, [userId])
+┊   ┊122┊
+┊   ┊123┊  const handleSignIn = useCallback(() => {
+┊   ┊124┊    signIn(userId).then(() => {
+┊   ┊125┊      history.replace('/chats')
+┊   ┊126┊    })
+┊   ┊127┊  }, [userId])
+┊   ┊128┊
+┊   ┊129┊  return (
+┊   ┊130┊    <Container>
+┊   ┊131┊      <Intro>
+┊   ┊132┊        <Icon src="assets/whatsapp-icon.png" className="AuthScreen-icon" />
+┊   ┊133┊        <Title className="AuthScreen-title">WhatsApp</Title>
+┊   ┊134┊      </Intro>
+┊   ┊135┊      <SignInForm>
+┊   ┊136┊        <ActualForm>
+┊   ┊137┊          <Legend>Sign in</Legend>
+┊   ┊138┊          <Section>
+┊   ┊139┊            <TextField
+┊   ┊140┊              data-testid="user-id-input"
+┊   ┊141┊              label="User ID"
+┊   ┊142┊              value={userId}
+┊   ┊143┊              onChange={onUserIdChange}
+┊   ┊144┊              margin="normal"
+┊   ┊145┊              placeholder="Enter current user ID"
+┊   ┊146┊            />
+┊   ┊147┊          </Section>
+┊   ┊148┊          <Button
+┊   ┊149┊            data-testid="sign-in-button"
+┊   ┊150┊            type="button"
+┊   ┊151┊            color="secondary"
+┊   ┊152┊            variant="contained"
+┊   ┊153┊            disabled={!maySignIn()}
+┊   ┊154┊            onClick={handleSignIn}
+┊   ┊155┊          >
+┊   ┊156┊            Sign in
+┊   ┊157┊          </Button>
+┊   ┊158┊        </ActualForm>
+┊   ┊159┊      </SignInForm>
+┊   ┊160┊    </Container>
+┊   ┊161┊  )
+┊   ┊162┊}
```

[}]: #

Accordingly we will define a new `/sign-in` route that will render the `AuthScreen` we’re under that path name:

[{]: <helper> (diffStep 11.4 files="App" module="client")

#### [Step 11.4: Add AuthScreen](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/647bb65)

##### Changed src&#x2F;App.jsx
```diff
@@ -1,5 +1,6 @@
 ┊1┊1┊import * as React from 'react'
 ┊2┊2┊import { BrowserRouter, Route, Redirect } from 'react-router-dom'
+┊ ┊3┊import AuthScreen from './components/AuthScreen'
 ┊3┊4┊import ChatRoomScreen from './components/ChatRoomScreen'
 ┊4┊5┊import ChatsListScreen from './components/ChatsListScreen'
 ┊5┊6┊import AnimatedSwitch from './components/AnimatedSwitch'
```
```diff
@@ -11,6 +12,7 @@
 ┊11┊12┊  return (
 ┊12┊13┊    <BrowserRouter>
 ┊13┊14┊      <AnimatedSwitch>
+┊  ┊15┊        <Route exact path="/sign-in" component={AuthScreen} />
 ┊14┊16┊        <Route exact path="/chats" component={ChatsListScreen} />
 ┊15┊17┊        <Route exact path="/chats/:chatId" component={ChatRoomScreen} />
 ┊16┊18┊      </AnimatedSwitch>
```

[}]: #

This is how the new screen should look like:

![auth-screen](https://user-images.githubusercontent.com/7648874/55606715-7a56a180-57ac-11e9-8eea-2da5931cccf5.png)

Now let’s type the `/sign-in` route in our browser’s navigation bar and assign a user ID, see how it affects what chats we see in the `ChatsListScreen`. You’ve probably noticed that there’s no way to escape from the `/chats` route unless we edit the browser’s navigation bar manually. To fix that, we will add a new sign-out button to the navbar of the `ChatsListScreen` that will call the `signOut()` method anytime we click on it, and will bring us back to the `AuthScreen`:

[{]: <helper> (diffStep 11.5 module="client")

#### [Step 11.5: Add sign-out button that ChatsNavbar](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/dc1711e)

##### Changed src&#x2F;components&#x2F;ChatsListScreen&#x2F;ChatsNavbar.tsx
```diff
@@ -1,18 +1,41 @@
-┊ 1┊  ┊import { Toolbar } from '@material-ui/core'
+┊  ┊ 1┊import { Button, Toolbar } from '@material-ui/core'
+┊  ┊ 2┊import SignOutIcon from '@material-ui/icons/PowerSettingsNew'
 ┊ 2┊ 3┊import * as React from 'react'
+┊  ┊ 4┊import { useCallback } from 'react'
 ┊ 3┊ 5┊import styled from 'styled-components'
+┊  ┊ 6┊import { signOut } from '../../services/auth.service'
 ┊ 4┊ 7┊
 ┊ 5┊ 8┊const Container = styled(Toolbar) `
+┊  ┊ 9┊  display: flex;
 ┊ 6┊10┊  background-color: var(--primary-bg);
 ┊ 7┊11┊  color: var(--primary-text);
 ┊ 8┊12┊  font-size: 20px;
 ┊ 9┊13┊  line-height: 40px;
 ┊10┊14┊`
 ┊11┊15┊
-┊12┊  ┊const ChatsNavbar = () => (
-┊13┊  ┊  <Container>
-┊14┊  ┊    Whatsapp Clone
-┊15┊  ┊  </Container>
-┊16┊  ┊)
+┊  ┊16┊const Title = styled.div `
+┊  ┊17┊  flex: 1;
+┊  ┊18┊`
+┊  ┊19┊
+┊  ┊20┊const LogoutButton = styled(Button) `
+┊  ┊21┊  color: var(--primary-text) !important;
+┊  ┊22┊`
+┊  ┊23┊
+┊  ┊24┊const ChatsNavbar = ({ history }) => {
+┊  ┊25┊  const handleSignOut = useCallback(() => {
+┊  ┊26┊    signOut().then(() => {
+┊  ┊27┊      history.replace('/sign-in')
+┊  ┊28┊    })
+┊  ┊29┊  }, [true])
+┊  ┊30┊
+┊  ┊31┊  return (
+┊  ┊32┊    <Container>
+┊  ┊33┊      <Title>Whatsapp Clone</Title>
+┊  ┊34┊      <LogoutButton data-testid="sign-out-button" onClick={handleSignOut}>
+┊  ┊35┊        <SignOutIcon />
+┊  ┊36┊      </LogoutButton>
+┊  ┊37┊    </Container>
+┊  ┊38┊  )
+┊  ┊39┊}
 ┊17┊40┊
-┊18┊  ┊export default ChatsNavbar
+┊  ┊41┊export default ChatsNavbar🚫↵
```

##### Changed src&#x2F;components&#x2F;ChatsListScreen&#x2F;index.tsx
```diff
@@ -9,7 +9,7 @@
 ┊ 9┊ 9┊
 ┊10┊10┊const ChatsListScreen = ({ history }) => (
 ┊11┊11┊  <Container>
-┊12┊  ┊    <ChatsNavbar />
+┊  ┊12┊    <ChatsNavbar history={history} />
 ┊13┊13┊    <ChatsList history={history} />
 ┊14┊14┊  </Container>
 ┊15┊15┊)
```

[}]: #

At this point we’ve got everything we need, but we will add a small touch to improve the user experience and make it feel more complete. Users who aren’t logged in shouldn’t be able to view any screen besides the `AuthScreen`. First they need to sign-in, and only then they will be able to view the `ChatsListScreen` and `ChatRoomScreen`. To achieve that, we will wrap all the components which require authentication before we provide them into their routes. This wrap will basically check whether a user is logged in or not by reading the cookies, and if not we will be redirected to the `/sign-in` route. Let’s implement that wrap in the `auth.service` and call it `withAuth()`:

[{]: <helper> (diffStep 11.6 files="auth.service" module="client")

#### [Step 11.6: Add withAuth() route wrapper](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/d622494)

##### Changed src&#x2F;services&#x2F;auth.service.tsx
```diff
@@ -1,4 +1,27 @@
+┊  ┊ 1┊import * as React from 'react'
+┊  ┊ 2┊import { Redirect } from 'react-router-dom'
 ┊ 1┊ 3┊import client from '../client'
+┊  ┊ 4┊import { useCacheService } from './cache.service'
+┊  ┊ 5┊
+┊  ┊ 6┊export const withAuth = (Component: React.ComponentType) => {
+┊  ┊ 7┊  return (props) => {
+┊  ┊ 8┊    if (!isSignedIn()) {
+┊  ┊ 9┊      if (props.history.location.pathname === '/sign-in') {
+┊  ┊10┊        return null
+┊  ┊11┊      }
+┊  ┊12┊
+┊  ┊13┊      return (
+┊  ┊14┊        <Redirect to="/sign-in" />
+┊  ┊15┊      )
+┊  ┊16┊    }
+┊  ┊17┊
+┊  ┊18┊    useCacheService()
+┊  ┊19┊
+┊  ┊20┊    return (
+┊  ┊21┊      <Component {...props} />
+┊  ┊22┊    )
+┊  ┊23┊  }
+┊  ┊24┊}
 ┊ 2┊25┊
 ┊ 3┊26┊export const signIn = (currentUserId) => {
 ┊ 4┊27┊  document.cookie = `currentUserId=${currentUserId}`
```

[}]: #

We will use this function to wrap the right components in our app’s router. Note that since we used the `useCacheService()` directly in the `withAuth()` method, there’s no need to use it in the router itself anymore. This makes a lot more sense since there’s no need to stay subscribed to data that you're not gonna receive from the first place unless you’re logged-in:

[{]: <helper> (diffStep 11.6 files="App" module="client")

#### [Step 11.6: Add withAuth() route wrapper](https://github.com/Urigo/WhatsApp-Clone-Client-React/commit/d622494)

##### Changed src&#x2F;App.jsx
```diff
@@ -4,22 +4,18 @@
 ┊ 4┊ 4┊import ChatRoomScreen from './components/ChatRoomScreen'
 ┊ 5┊ 5┊import ChatsListScreen from './components/ChatsListScreen'
 ┊ 6┊ 6┊import AnimatedSwitch from './components/AnimatedSwitch'
-┊ 7┊  ┊import { useCacheService } from './services/cache.service'
+┊  ┊ 7┊import { withAuth } from './services/auth.service'
 ┊ 8┊ 8┊
-┊ 9┊  ┊const App = () => {
-┊10┊  ┊  useCacheService()
-┊11┊  ┊
-┊12┊  ┊  return (
-┊13┊  ┊    <BrowserRouter>
-┊14┊  ┊      <AnimatedSwitch>
-┊15┊  ┊        <Route exact path="/sign-in" component={AuthScreen} />
-┊16┊  ┊        <Route exact path="/chats" component={ChatsListScreen} />
-┊17┊  ┊        <Route exact path="/chats/:chatId" component={ChatRoomScreen} />
-┊18┊  ┊      </AnimatedSwitch>
-┊19┊  ┊      <Route exact path="/" render={redirectToChats} />
-┊20┊  ┊    </BrowserRouter>
-┊21┊  ┊  )
-┊22┊  ┊}
+┊  ┊ 9┊const App = () => (
+┊  ┊10┊  <BrowserRouter>
+┊  ┊11┊    <AnimatedSwitch>
+┊  ┊12┊      <Route exact path="/sign-in" component={AuthScreen} />
+┊  ┊13┊      <Route exact path="/chats" component={withAuth(ChatsListScreen)} />
+┊  ┊14┊      <Route exact path="/chats/:chatId" component={withAuth(ChatRoomScreen)} />
+┊  ┊15┊    </AnimatedSwitch>
+┊  ┊16┊    <Route exact path="/" render={redirectToChats} />
+┊  ┊17┊  </BrowserRouter>
+┊  ┊18┊)
 ┊23┊19┊
 ┊24┊20┊const redirectToChats = () => (
 ┊25┊21┊  <Redirect to="/chats" />
```

[}]: #

Assuming that you’re not logged-in, if you’ll try to force navigate to the `/chats` route you should be automatically redirected to the `/sign-in` form. We will finish the chapter here as we wanna keep things simple and gradual. It’s true that we haven’t implemented true authentication, but that would be addressed soon further in this tutorial.


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](https://github.com/Urigo/WhatsApp-Clone-Tutorial/tree/final@next/.tortilla/manuals/views/step10.md) | [Next Step >](https://github.com/Urigo/WhatsApp-Clone-Tutorial/tree/final@next/.tortilla/manuals/views/step12.md) |
|:--------------------------------|--------------------------------:|

[}]: #
