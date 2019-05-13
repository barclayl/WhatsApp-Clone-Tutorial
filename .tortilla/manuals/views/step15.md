# Step 15: Using a REST API

[//]: # (head-end)


Despite using GraphQL throughout all our app, we will soon meet the need to use some external API and chances are it will be REST.
Our first idea could be to bridge the REST API through GraphQL, reproposing the very same API to the client. This approach is wrong, because our first concern should always be to provide the client with ready to use data in the best possible shape.
The client don’t need to know that our GraphQL API is backed by a REST API, it doesn’t have to pass headers required by the underlying API or do any kind of special considerations: our backend should take care of everything.

## Retrieve a profile picture from a REST API

In this chapter we will discuss how to use an external API called Unsplash to retrieve random profile pictures for the users who didn’t set any.

Start heading to https://unsplash.com/developers and clicking on “Register as a developer”. After registering you will have to create a new app: take note of the Access Key because we’re going to need it.

If you look at the Documentation (https://unsplash.com/documentation#get-a-random-photo) you’ll notice that in order to retrieve a random photo we have to query the /photos/random endpoint (GET method). We also have to pass some headers for the authent
cation and some params for the search term and the orientation.

On the browser we would probably use the fetch api, but since on we node we would need a polyfill it’s better to just use a full fledged library like axios:

    yarn add axios
    yarn add -D @types/axios

Before we start implementing, we want to create some typings for our endpoint, because ideally we would like to be aided by those typings during the development.
In order to do so we can use a Chrome extension like Advanced Rest Client to retrieve the response.
Set the Method to GET, the Headers to Authorization: 'Client-ID 4d048cfb4383b407eff92e4a2a5ec36c0a866be85e64caafa588c110efad350d' and the Request URL to https://api.unsplash.com/photos/random, along with the params to query: 'portrait' and orientation: 'squarish'.
Copy the response, create a new file called types/unsplash.ts in your vscode editor and run the command “Past JSON as Types” (you need to install the Past JSON as Code extension and press CTRL+P to open the run command prompt). That would be enough to automatically create the typings for the random photo endpoint.

Now we can finally implement the REST API call in our picture resolver:

[{]: <helper> (diffStep "12.1" files="schema/resolvers.ts" module="server")

#### Step 12.1: Retrieve profile picture from REST API

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -7,6 +7,8 @@
 ┊ 7┊ 7┊import jwt from 'jsonwebtoken'
 ┊ 8┊ 8┊import { validateLength, validatePassword } from '../validators';
 ┊ 9┊ 9┊import sql from 'sql-template-strings'
+┊  ┊10┊import axios from 'axios'
+┊  ┊11┊import { RandomPhoto } from "../types/unsplash"
 ┊10┊12┊
 ┊11┊13┊const resolvers: Resolvers = {
 ┊12┊14┊  Date: GraphQLDateTime,
```
```diff
@@ -70,7 +72,22 @@
 ┊70┊72┊
 ┊71┊73┊      const participant = rows[0]
 ┊72┊74┊
-┊73┊  ┊      return participant ? participant.picture : null
+┊  ┊75┊      if (participant && participant.picture) return participant.picture
+┊  ┊76┊
+┊  ┊77┊      try {
+┊  ┊78┊        return (await axios.get<RandomPhoto>('https://api.unsplash.com/photos/random', {
+┊  ┊79┊          params: {
+┊  ┊80┊            query: 'portrait',
+┊  ┊81┊            orientation: 'squarish',
+┊  ┊82┊          },
+┊  ┊83┊          headers: {
+┊  ┊84┊            Authorization: 'Client-ID 4d048cfb4383b407eff92e4a2a5ec36c0a866be85e64caafa588c110efad350d'
+┊  ┊85┊          },
+┊  ┊86┊        })).data.urls.small
+┊  ┊87┊      } catch (err) {
+┊  ┊88┊        console.error('Cannot retrieve random photo:', err)
+┊  ┊89┊        return null
+┊  ┊90┊      }
 ┊74┊91┊    },
 ┊75┊92┊
 ┊76┊93┊    async messages(chat, args, { db }) {
```
```diff
@@ -126,7 +143,7 @@
 ┊126┊143┊        AND chats.id = chats_users.chat_id
 ┊127┊144┊        AND chats_users.user_id = ${currentUser.id}
 ┊128┊145┊      `)
-┊129┊   ┊
+┊   ┊146┊
 ┊130┊147┊      return rows[0] ? rows[0] : null
 ┊131┊148┊    },
 ┊132┊149┊
```
```diff
@@ -163,31 +180,31 @@
 ┊163┊180┊
 ┊164┊181┊      return user;
 ┊165┊182┊    },
-┊166┊   ┊
+┊   ┊183┊
 ┊167┊184┊    async signUp(root, { name, username, password, passwordConfirm }, { db }) {
 ┊168┊185┊      validateLength('req.name', name, 3, 50)
 ┊169┊186┊      validateLength('req.username', name, 3, 18)
 ┊170┊187┊      validatePassword('req.password', password)
-┊171┊   ┊
+┊   ┊188┊
 ┊172┊189┊      if (password !== passwordConfirm) {
 ┊173┊190┊        throw Error("req.password and req.passwordConfirm don't match")
 ┊174┊191┊      }
-┊175┊   ┊
+┊   ┊192┊
 ┊176┊193┊      const existingUserQuery = await db.query(sql`SELECT * FROM users WHERE username = ${username}`)
 ┊177┊194┊      if (existingUserQuery.rows[0]) {
 ┊178┊195┊        throw Error("username already exists")
 ┊179┊196┊      }
-┊180┊   ┊
+┊   ┊197┊
 ┊181┊198┊      const passwordHash = bcrypt.hashSync(password, bcrypt.genSaltSync(8))
-┊182┊   ┊
+┊   ┊199┊
 ┊183┊200┊      const createdUserQuery = await db.query(sql`
 ┊184┊201┊        INSERT INTO users(password, picture, username, name)
 ┊185┊202┊        VALUES(${passwordHash}, '', ${username}, ${name})
 ┊186┊203┊        RETURNING *
 ┊187┊204┊      `)
-┊188┊   ┊
+┊   ┊205┊
 ┊189┊206┊      const user = createdUserQuery.rows[0]
-┊190┊   ┊
+┊   ┊207┊
 ┊191┊208┊      return user;
 ┊192┊209┊
 ┊193┊210┊    },
```

[}]: #

In order to test it, we have to remove the picture from one of the users and re-run the server with the `RESET_DB=true` environment variable:

[{]: <helper> (diffStep "12.1" files="db.ts" module="server")

#### Step 12.1: Retrieve profile picture from REST API

##### Changed db.ts
```diff
@@ -40,7 +40,7 @@
 ┊40┊40┊      name: 'Ray Edwards',
 ┊41┊41┊      username: 'ray',
 ┊42┊42┊      password: '$2a$08$NO9tkFLCoSqX1c5wk3s7z.JfxaVMKA.m7zUDdDwEquo4rvzimQeJm', // 111
-┊43┊  ┊      picture: 'https://randomuser.me/api/portraits/thumb/lego/1.jpg',
+┊  ┊43┊      picture: '',
 ┊44┊44┊    },
 ┊45┊45┊    {
 ┊46┊46┊      id: '2',
```
```diff
@@ -71,14 +71,14 @@
 ┊71┊71┊      picture: 'https://randomuser.me/api/portraits/thumb/women/2.jpg',
 ┊72┊72┊    },
 ┊73┊73┊  ]
-┊74┊  ┊
+┊  ┊74┊
 ┊75┊75┊  for (const sampleUser of sampleUsers) {
 ┊76┊76┊    await pool.query(sql`
 ┊77┊77┊      INSERT INTO users(id, name, username, password, picture)
 ┊78┊78┊      VALUES(${sampleUser.id}, ${sampleUser.name}, ${sampleUser.username}, ${sampleUser.password}, ${sampleUser.picture})
 ┊79┊79┊    `)
 ┊80┊80┊  }
-┊81┊  ┊
+┊  ┊81┊
 ┊82┊82┊  await pool.query(sql`SELECT setval('users_id_seq', (SELECT max(id) FROM users))`)
 ┊83┊83┊
 ┊84┊84┊  await pool.query(sql`DELETE FROM chats`)
```

[}]: #


## Track the API

Even if our typings are working pretty well so far, not all REST APIs are versioned and the shape we’ve got from the server could potentially change.
In order to keep an eye on it we could use the safe-api middleware in order to check for abnormal answers coming from the server and log them. We can also generate the typings automatically based on the response we get.
First let’s install the safe-api middleware:

    yarn add @safe-api/middleware

Then let’s use it inside our resolver:

[{]: <helper> (diffStep "12.2" files="schema/resolvers.ts" module="server")

#### Step 12.2: Use safe-api

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -9,6 +9,8 @@
 ┊ 9┊ 9┊import sql from 'sql-template-strings'
 ┊10┊10┊import axios from 'axios'
 ┊11┊11┊import { RandomPhoto } from "../types/unsplash"
+┊  ┊12┊import { trackProvider } from '@safe-api/middleware'
+┊  ┊13┊import { resolve } from "path"
 ┊12┊14┊
 ┊13┊15┊const resolvers: Resolvers = {
 ┊14┊16┊  Date: GraphQLDateTime,
```
```diff
@@ -74,16 +76,32 @@
 ┊ 74┊ 76┊
 ┊ 75┊ 77┊      if (participant && participant.picture) return participant.picture
 ┊ 76┊ 78┊
+┊   ┊ 79┊      interface RandomPhotoInput {
+┊   ┊ 80┊        query: string;
+┊   ┊ 81┊        orientation: 'landscape' | 'portrait' | 'squarish'
+┊   ┊ 82┊      }
+┊   ┊ 83┊
+┊   ┊ 84┊      const trackedRandomPhoto = await trackProvider(async ({query, orientation}: RandomPhotoInput) =>
+┊   ┊ 85┊          (await axios.get<RandomPhoto>('https://api.unsplash.com/photos/random', {
+┊   ┊ 86┊            params: {
+┊   ┊ 87┊              query,
+┊   ┊ 88┊              orientation,
+┊   ┊ 89┊            },
+┊   ┊ 90┊            headers: {
+┊   ┊ 91┊              Authorization: 'Client-ID 4d048cfb4383b407eff92e4a2a5ec36c0a866be85e64caafa588c110efad350d'
+┊   ┊ 92┊            },
+┊   ┊ 93┊          })).data,
+┊   ┊ 94┊        {
+┊   ┊ 95┊          provider: 'Unsplash',
+┊   ┊ 96┊          method: 'RandomPhoto',
+┊   ┊ 97┊          location: resolve(__dirname, '../logs/main'),
+┊   ┊ 98┊        })
+┊   ┊ 99┊
 ┊ 77┊100┊      try {
-┊ 78┊   ┊        return (await axios.get<RandomPhoto>('https://api.unsplash.com/photos/random', {
-┊ 79┊   ┊          params: {
-┊ 80┊   ┊            query: 'portrait',
-┊ 81┊   ┊            orientation: 'squarish',
-┊ 82┊   ┊          },
-┊ 83┊   ┊          headers: {
-┊ 84┊   ┊            Authorization: 'Client-ID 4d048cfb4383b407eff92e4a2a5ec36c0a866be85e64caafa588c110efad350d'
-┊ 85┊   ┊          },
-┊ 86┊   ┊        })).data.urls.small
+┊   ┊101┊        return (await trackedRandomPhoto({
+┊   ┊102┊          query: 'portrait',
+┊   ┊103┊          orientation: 'squarish',
+┊   ┊104┊        })).urls.small
 ┊ 87┊105┊      } catch (err) {
 ┊ 88┊106┊        console.error('Cannot retrieve random photo:', err)
 ┊ 89┊107┊        return null
```

[}]: #

Now launch the client in order to retrieve the picture field multiple times.

If you look inside the logs directory you will notice that it generated some graphql schema to represent the REST API. You will notice that each time we call the REST endpoint it generates a new schema, because a single response isn’t generic enough to account for all possible responses. Ideally safe-api should be able to average multiple esponses in order to generate the least generic schema matching the given responses.

Now we need to remove `types/unsplash.ts` and generate some Typescript typings out of the schema. Do do so we can use the graphql-code-generator:

[{]: <helper> (diffStep "12.3" files=".gitignore, codegen.yml" module="server")

#### Step 12.3: Generate typings from safe-api

##### Changed .gitignore
```diff
@@ -1,3 +1,4 @@
 ┊1┊1┊node_modules
 ┊2┊2┊npm-debug.log
-┊3┊ ┊types/graphql.d.ts🚫↵
+┊ ┊3┊types/graphql.d.ts
+┊ ┊4┊types/unsplash.d.ts
```

##### Changed codegen.yml
```diff
@@ -1,7 +1,7 @@
-┊1┊ ┊schema: ./schema/typeDefs.graphql
 ┊2┊1┊overwrite: true
 ┊3┊2┊generates:
 ┊4┊3┊  ./types/graphql.d.ts:
+┊ ┊4┊    schema: ./schema/typeDefs.graphql
 ┊5┊5┊    plugins:
 ┊6┊6┊      - typescript
 ┊7┊7┊      - typescript-resolvers
```
```diff
@@ -16,3 +16,7 @@
 ┊16┊16┊      scalars:
 ┊17┊17┊        # e.g. Message.createdAt will be of type Date
 ┊18┊18┊        Date: Date
+┊  ┊19┊  ./types/unsplash.d.ts:
+┊  ┊20┊    schema: ./logs/main/Unsplash.RandomPhoto.graphql
+┊  ┊21┊    plugins:
+┊  ┊22┊      - typescript
```

[}]: #

    yarn codegen


## Apollo DataSources

We’re not done yet, there is still room for improvement. Instead of using axios, we could use Apollo’s Data Sources and take advantage of the built-in support for caching, deduplication and error handling.

    yarn remove axios @types/axios
    yarn add apollo-datasource-rest

[{]: <helper> (diffStep "12.4" files="schema/unsplash.api.ts" module="server")

#### Step 12.4: Use Apollo DataSources

##### Added schema&#x2F;unsplash.api.ts
```diff
@@ -0,0 +1,40 @@
+┊  ┊ 1┊import { RESTDataSource, RequestOptions } from "apollo-datasource-rest";
+┊  ┊ 2┊import { resolve } from "path";
+┊  ┊ 3┊import { trackProvider } from '@safe-api/middleware';
+┊  ┊ 4┊import { RandomPhoto } from "../types/unsplash";
+┊  ┊ 5┊
+┊  ┊ 6┊interface RandomPhotoInput {
+┊  ┊ 7┊  query: string
+┊  ┊ 8┊  orientation: 'landscape' | 'portrait' | 'squarish'
+┊  ┊ 9┊}
+┊  ┊10┊
+┊  ┊11┊export class UnsplashApi extends RESTDataSource {
+┊  ┊12┊  constructor() {
+┊  ┊13┊    super();
+┊  ┊14┊    this.baseURL = 'https://api.unsplash.com/'
+┊  ┊15┊  }
+┊  ┊16┊
+┊  ┊17┊  willSendRequest(request: RequestOptions) {
+┊  ┊18┊    request.headers.set('Authorization', 'Client-ID 4d048cfb4383b407eff92e4a2a5ec36c0a866be85e64caafa588c110efad350d')
+┊  ┊19┊  }
+┊  ┊20┊
+┊  ┊21┊  async getRandomPhoto() {
+┊  ┊22┊    const trackedRandomPhoto = await trackProvider(({query, orientation}: RandomPhotoInput) =>
+┊  ┊23┊        this.get<RandomPhoto>('photos/random', {query, orientation}),
+┊  ┊24┊      {
+┊  ┊25┊        provider: 'Unsplash',
+┊  ┊26┊        method: 'RandomPhoto',
+┊  ┊27┊        location: resolve(__dirname, '../logs/main'),
+┊  ┊28┊      })
+┊  ┊29┊
+┊  ┊30┊    try {
+┊  ┊31┊      return (await trackedRandomPhoto({
+┊  ┊32┊        query: 'portrait',
+┊  ┊33┊        orientation: 'squarish',
+┊  ┊34┊      })).urls.small
+┊  ┊35┊    } catch (err) {
+┊  ┊36┊      console.error('Cannot retrieve random photo:', err)
+┊  ┊37┊      return null
+┊  ┊38┊    }
+┊  ┊39┊  }
+┊  ┊40┊}
```

[}]: #

We created the UnsplashApi class, which extends RESTDataSource. In the constructor you need to set the baseUrl (after calling super() to run the constructor of the base class). You also need to create a willSendRequest method to set the authentication headers for each call. Then it’s simply a matter of creating a getRandomPhoto method to perform the actual REST API call. Instead of calling axios you will have to call the get method of the class (which in turn gets inherited from its RESTDataSource base class): the API is very similar to the axios one.

In order to access the data source from the resolvers we need to tell Apollo to put them on the context for every request. We shouldn’t use the context field, because that would lead to circular dependencies. Instead we need to use the dataSources field:

[{]: <helper> (diffStep "12.4" files="index.ts" module="server")

#### Step 12.4: Use Apollo DataSources

##### Changed index.ts
```diff
@@ -7,6 +7,7 @@
 ┊ 7┊ 7┊import schema from './schema'
 ┊ 8┊ 8┊import { MyContext } from './context';
 ┊ 9┊ 9┊import sql from 'sql-template-strings'
+┊  ┊10┊import { UnsplashApi } from "./schema/unsplash.api";
 ┊10┊11┊const { PostgresPubSub } = require('graphql-postgres-subscriptions')
 ┊11┊12┊
 ┊12┊13┊const pubsub = new PostgresPubSub({
```
```diff
@@ -24,7 +25,7 @@
 ┊24┊25┊    if(!connection) {
 ┊25┊26┊      db = await pool.connect();
 ┊26┊27┊    }
-┊27┊  ┊
+┊  ┊28┊
 ┊28┊29┊    let currentUser
 ┊29┊30┊    if (req.cookies.authToken) {
 ┊30┊31┊      const username = jwt.verify(req.cookies.authToken, secret) as string
```
```diff
@@ -44,7 +45,10 @@
 ┊44┊45┊    context.db.release()
 ┊45┊46┊
 ┊46┊47┊    return res
-┊47┊  ┊  }
+┊  ┊48┊  },
+┊  ┊49┊  dataSources: () => ({
+┊  ┊50┊    unsplashApi: new UnsplashApi(),
+┊  ┊51┊  }),
 ┊48┊52┊})
 ┊49┊53┊
 ┊50┊54┊server.applyMiddleware({
```

[}]: #

Now we need to update the typings for our context and run the graphq-code-generator again:

[{]: <helper> (diffStep "12.4" files="context.ts" module="server")

#### Step 12.4: Use Apollo DataSources

##### Changed context.ts
```diff
@@ -2,10 +2,14 @@
 ┊ 2┊ 2┊import { User } from './db'
 ┊ 3┊ 3┊import { Response } from 'express'
 ┊ 4┊ 4┊import { PoolClient } from 'pg';
+┊  ┊ 5┊import { UnsplashApi } from "./schema/unsplash.api";
 ┊ 5┊ 6┊
 ┊ 6┊ 7┊export type MyContext = {
 ┊ 7┊ 8┊  pubsub: PubSub,
 ┊ 8┊ 9┊  currentUser: User,
 ┊ 9┊10┊  res: Response,
 ┊10┊11┊  db: PoolClient,
+┊  ┊12┊  dataSources: {
+┊  ┊13┊    unsplashApi: UnsplashApi,
+┊  ┊14┊  },
 ┊11┊15┊}
```

[}]: #

    yarn codegen

Now it should be pretty easy to modify our resolver in order to use our just created datasource:

[{]: <helper> (diffStep "12.4" files="schema/resolvers.ts" module="server")

#### Step 12.4: Use Apollo DataSources

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -7,10 +7,6 @@
 ┊ 7┊ 7┊import jwt from 'jsonwebtoken'
 ┊ 8┊ 8┊import { validateLength, validatePassword } from '../validators';
 ┊ 9┊ 9┊import sql from 'sql-template-strings'
-┊10┊  ┊import axios from 'axios'
-┊11┊  ┊import { RandomPhoto } from "../types/unsplash"
-┊12┊  ┊import { trackProvider } from '@safe-api/middleware'
-┊13┊  ┊import { resolve } from "path"
 ┊14┊10┊
 ┊15┊11┊const resolvers: Resolvers = {
 ┊16┊12┊  Date: GraphQLDateTime,
```
```diff
@@ -63,7 +59,7 @@
 ┊63┊59┊      return participant ? participant.name : null
 ┊64┊60┊    },
 ┊65┊61┊
-┊66┊  ┊    async picture(chat, args, { currentUser, db }) {
+┊  ┊62┊    async picture(chat, args, { currentUser, db, dataSources }) {
 ┊67┊63┊      if (!currentUser) return null
 ┊68┊64┊
 ┊69┊65┊      const { rows } = await db.query(sql`
```
```diff
@@ -74,38 +70,7 @@
 ┊ 74┊ 70┊
 ┊ 75┊ 71┊      const participant = rows[0]
 ┊ 76┊ 72┊
-┊ 77┊   ┊      if (participant && participant.picture) return participant.picture
-┊ 78┊   ┊
-┊ 79┊   ┊      interface RandomPhotoInput {
-┊ 80┊   ┊        query: string;
-┊ 81┊   ┊        orientation: 'landscape' | 'portrait' | 'squarish'
-┊ 82┊   ┊      }
-┊ 83┊   ┊
-┊ 84┊   ┊      const trackedRandomPhoto = await trackProvider(async ({query, orientation}: RandomPhotoInput) =>
-┊ 85┊   ┊          (await axios.get<RandomPhoto>('https://api.unsplash.com/photos/random', {
-┊ 86┊   ┊            params: {
-┊ 87┊   ┊              query,
-┊ 88┊   ┊              orientation,
-┊ 89┊   ┊            },
-┊ 90┊   ┊            headers: {
-┊ 91┊   ┊              Authorization: 'Client-ID 4d048cfb4383b407eff92e4a2a5ec36c0a866be85e64caafa588c110efad350d'
-┊ 92┊   ┊            },
-┊ 93┊   ┊          })).data,
-┊ 94┊   ┊        {
-┊ 95┊   ┊          provider: 'Unsplash',
-┊ 96┊   ┊          method: 'RandomPhoto',
-┊ 97┊   ┊          location: resolve(__dirname, '../logs/main'),
-┊ 98┊   ┊        })
-┊ 99┊   ┊
-┊100┊   ┊      try {
-┊101┊   ┊        return (await trackedRandomPhoto({
-┊102┊   ┊          query: 'portrait',
-┊103┊   ┊          orientation: 'squarish',
-┊104┊   ┊        })).urls.small
-┊105┊   ┊      } catch (err) {
-┊106┊   ┊        console.error('Cannot retrieve random photo:', err)
-┊107┊   ┊        return null
-┊108┊   ┊      }
+┊   ┊ 73┊      return (participant && participant.picture) ? participant.picture : dataSources.unsplashApi.getRandomPhoto()
 ┊109┊ 74┊    },
 ┊110┊ 75┊
 ┊111┊ 76┊    async messages(chat, args, { db }) {
```

[}]: #


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](https://github.com/Urigo/WhatsApp-Clone-Tutorial/tree/final@next/.tortilla/manuals/views/step14.md) |
|:----------------------|

[}]: #
