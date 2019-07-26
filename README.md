Quick Start: Amplify Primer
===

This quick start is intended to showcase what you can do with
the [AWS Amplify Framework][1].

The Amplify Framework makes it easier to integrate your web and 
mobile apps (iOS, Android, React Native, etc) to managed AWS
services.

[More detailed instructions here][2]

---

## Prerequisites

### AWS account

You will need an AWS account to deploy resources using the 
Amplify Framework. If you are in an instructor-led event or
workshop, confirm with your facilitator if they have
an AWS account you can use.

### Node.js

The Amplify Framework CLI tool runs on Node.js, so your 
workspace will need to have it installed. If you don't have it
installed yet, we recommend you use a version manager like
[nvm][3].

---

## Instructions

### Prepare your baseline app

These steps walk you through creating a sample app that 
we will integrate the Amplify framework to.

1. Install the Amplify CLI.

   ```bash
   npm install -g @aws-amplify/cli
   ```

2. Configure the AWS profile that the Amplify CLI will use.

   ```bash
   amplify configure
   ```

   > Running this command will walk you through a wizard for creating
   > an IAM user that Amplify will use. Follow the steps, 
   > providing your preferred username and region.

3. Create a folder for your app, and initialize your files.

   ```bash
   mkdir -p quickstart-app/src && cd quickstart-app

   touch \
     package.json \
     index.html \
     webpack.config.js \
     src/app.js
   ```

   This will create a directory containing the following
   structure:

   ```
   - quickstart-app
     - index.html
     - package.json
     - webpack.config.js
     - /src
       - app.js
   ```

4. Add the following contents to `package.json`:

   ```json
   {
     "name": "quickstart-amplify-app",
     "version": "1.0.0",
     "description": "Amplify JavaScript Example",
     "dependencies": {
       "@aws-amplify/api": "latest",
       "@aws-amplify/pubsub": "latest"
     },
     "devDependencies": {
       "webpack": "^4.17.1",
       "webpack-cli": "^3.1.0",
       "copy-webpack-plugin": "^4.5.2",
       "webpack-dev-server": "^3.1.5"
     },
     "scripts": {
       "start": "webpack && webpack-dev-server --mode development",
       "build": "webpack"
     }
   }
   ```

   Afterwards, install the application dependencies we
   just defined:

   ```bash
   npm install
   ```

5. Add the following contents to `index.html`:

   ```html
   <!DOCTYPE html>
   <html lang="en">
       <head>
           <meta charset="utf-8">
           <title>Amplify Framework</title>
           <meta name="viewport" content="width=device-width, initial-scale=1">
           <style>
               html, body { font-family: "Amazon Ember", "Helvetica", "sans-serif"; margin: 0; }
               a { color: #FF9900; }
               h1 { font-weight: 300; }
               .app { width: 100%; }
               .app-header { color: white; text-align: center; background: linear-gradient(30deg, #f90 55%, #FFC300); width: 100%; margin: 0 0 1em 0; padding: 3em 0 3em 0; box-shadow: 1px 2px 4px rgba(0, 0, 0, .3); }
               .app-logo { width: 126px; margin: 0 auto; }
               .app-body { width: 400px; margin: 0 auto; text-align: center; }
               .app-body button { background-color: #FF9900; font-size: 14px; color: white; text-transform: uppercase; padding: 1em; border: none; }
               .app-body button:hover { opacity: 0.8; }
           </style>
       </head>
       <body>
           <div class="app">
               <div class="app-header">
                   <div class="app-logo">
                       <img src="https://aws-amplify.github.io/images/Logos/Amplify-Logo-White.svg" alt="AWS Amplify" />
                   </div>
                   <h1>Welcome to the Amplify Framework</h1>
               </div>
               <div class="app-body">
                   <button id="MutationEventButton">Add data</button>
                   <div id="MutationResult"></div>
                   <div id="QueryResult"></div>
                   <div id="SubscriptionResult"></div>
               </div>
           </div>
           <script src="main.bundle.js"></script>
       </body>
   </html>
   ```

6. Add the following contents to `webpack.config.js`:

   ```javascript
   const CopyWebpackPlugin = require('copy-webpack-plugin');
   const webpack = require('webpack');
   const path = require('path');

   module.exports = {
       mode: 'development',
       entry: './src/app.js',
       output: {
           filename: '[name].bundle.js',
           path: path.resolve(__dirname, 'dist')
       },
       module: {
           rules: [
               {
                   test: /\.js$/,
                   exclude: /node_modules/
               }
           ]
       },
       devServer: {
           contentBase: './dist',
           overlay: true,
           hot: true
       },
       plugins: [
           new CopyWebpackPlugin(['index.html']),
           new webpack.HotModuleReplacementPlugin()
       ]
   };
   ```

7. Run the app.

   ```bash
   npm start
   ```
  
   Open a browser and navigate to http://localhost:8080.
   The `Add data` button does not work yet.
   We'll work on that next.


### Set up the app backend

1. Initialize Amplify

   ```bash
   amplify init
   ```

   Walk through the steps, and verify that Amplify has been
   integrated by running:

   ```bash
   amplify status
   ```

   Amplify keeps a record of the AWS resources it will need
   to deploy and manage for your app. Right now there aren't
   any yet, so the table should be empty.

2. Add an API and Database

   We could add a database and the corresponding API to access
   it to our app manually, but we could also use Amplify to
   automatically add in the components required to our app and
   backend.

   Let's use Amplify to add in a GraphQL API 
   (using AWS AppSync) that will talk to
   an Amazon DynamoDB table.

   ```bash
   amplify add api
   ```

   For this exercise, just choose **No**, and let it guide you
   through the default project **Single object with fields
   (e.g. "Todo" with ID, name, description)**. Later on we can
   always change the resulting schema.

3. Deploy our changes

   ```bash
   amplify push
   ```

   This command will instruct Amplify to deploy the necessary
   resources to your AWS account. It's intelligent enough to 
   determine if a resource is something completely new, or if 
   only modifications are required to an existing resource.

   If if prompts you for automatic query and code generation,
   opt to use **Javascript** for this tutorial.


## Integrate into the app

Now that we have a simple backend using AWS resources,
let's work on integrating those into our app.

1. Update `src/app.js` with the following:

   ```javascript
   import API, { graphqlOperation } from '@aws-amplify/api'
   import PubSub from '@aws-amplify/pubsub';
   import { createTodo } from './graphql/mutations'

   import awsconfig from './aws-exports';
   API.configure(awsconfig);
   PubSub.configure(awsconfig);

   async function createNewTodo() {
     const todo = { name: "Use AppSync" , description: "Realtime and Offline"}
     return await API.graphql(graphqlOperation(createTodo, { input: todo }))
   }

   const MutationButton = document.getElementById('MutationEventButton');
   const MutationResult = document.getElementById('MutationResult');

   MutationButton.addEventListener('click', (evt) => {
     MutationResult.innerHTML = `MUTATION RESULTS:`;
     createNewTodo().then( (evt) => {
       MutationResult.innerHTML += `<p>${evt.data.createTodo.name} - ${evt.data.createTodo.description}</p>`
     })
   });
   ```

2. Start the app again:

   ```bash
   npm start
   ```

   Then browse to the app at http://localhost:8080. 
   Click the `Add data` button.

   At this point, you'll see the application now submitting
   events to AppSync and storing records in DynamoDB.

   We'll now update our app so that it can display the records
   currently in DynamoDB.

3. Update `src/app.js` again by adding the following content:

   ```javascript
   import { listTodos } from './graphql/queries'

   const QueryResult = document.getElementById('QueryResult');

   async function getData() {
     QueryResult.innerHTML = `QUERY RESULTS`;
     API.graphql(graphqlOperation(listTodos)).then((evt) => {
       evt.data.listTodos.items.map((todo, i) => 
       QueryResult.innerHTML += `<p>${todo.name} - ${todo.description}</p>`
       );
     })
   }

   getData();
   ```   

4. Add a subscription to your `src/app.js` as well:

   ```javascript
   // other imports
   import { onCreateTodo } from './graphql/subscriptions'

   const SubscriptionResult = document.getElementById('SubscriptionResult');

   API.graphql(graphqlOperation(onCreateTodo)).subscribe({
     next: (evt) =>{
       SubscriptionResult.innerHTML = `SUBSCRIPTION RESULTS`
       const todo = evt.value.data.onCreateTodo;
       SubscriptionResult.innerHTML += `<p>${todo.name} - ${todo.description}</p>`
     }
   });
   ```

   Don't forget to start the app again.

   ```bash
   npm start
   ```


## Launch your app

Let's use Amplify to deploy our entire app as a static web site
over Amazon S3.

1. Update our backend to use hosting resources.

   ```bash
   amplify add hosting
   ```

2. Then, publish our app.

   ```bash
   amplify publish
   ```

At this point, you can open the app and push the button
to generate new items in your database.

---
[1]: https://aws-amplify.github.io

[2]: https://aws-amplify.github.io/docs/js/start?ref=amplify-js-btn&platform=purejs

[3]: https://github.com/nvm-sh/nvm