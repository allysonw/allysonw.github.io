---
layout: post
title:      "Getting Started on a React App with a Rails API"
date:       2018-06-20 14:15:13 -0400
permalink:  getting_started_on_a_react_app_with_a_rails_api
---


When I first approached creating my final project for the Flatiron School's Full Stack Web Development program, I wasn't sure exactly where to start. I knew I need to create a Rails backend API to send data to and receive data from a React frontend client, but I didn't know in what order I should create the separate stacks, or how to design the overall architecture of the project. Thankfully, with the help of the resources I mention below, I was able to get up and running fairly quickly. This post is a guide for how to get started on the project.

### Step 1: rails new
Presuming you have a basic idea of what your app will do (as that should probably come first), the first step to get it going is to create the Rails API backend.

```$ rails new word-scramble --api --database=postgresql```

This tells Rails you want to create an app without any views (since you're building an API), and that you want the database to be PostgreSQL. For deploying to Heroku, you must use Postgres rather than SQLite, so if you're thinking ahead to deploying your project on that site, you'll want to start out using Postgres.

### Step 2: Set Up Your Backend
At this point, you can add the migrations, models, controllers, and routes that you will need in order to provide data to your frontend. Run your migrations to seed your DB. You'll want to add [serializers](https://learn.co/tracks/full-stack-web-development-v5/rails-and-javascript/building-apis/using-active-model-serializer) to get your data into a format where it can be sent to the frontend via HTTP responses. Also, set up at least one API endpoint for your frontend to access so you can verify both stacks are talking to each other, e.g.

```
# routes.rb
resources :games, only: :index
```

Make sure you have a controller action for that route, e.g.

```
# games_controller.rb
def index
  # Return all games to the client
  games = Game.all
  render json: games
end
```

Check that your server works and responds as you expect by running `rails s`.

### Step 3: create-react-app

`create-react-app` will set up a new React app for you, much like `rails new` does for Rails. There's still some configuration to be done afterwards, but first make sure you have `create-react-app` installed with:

```$ npm i -g create-react-app```

At this point you have a choice of whether to create your react app in a separate repository or in the same one as your rails app. There are benefits to both approaches, but I went with just creating a `client` folder in my Rails app, within which I wrote my React app.

```$ create-react-app client ```

`cd` into your `client` folder and go ahead and install the various dependencies React will need for your project.

```
$ npm install --save name-of-dependency
```

You'll need at least the following dependencies for this project:
```
react-redux
react-router-dom
redux
redux-thunk
```

You should now be able to run your React app using `$ npm start` and navigating to `localhost:3000` in your browser.

At this point, you should have verified that both your React app and Rails API can run on successfully on their respective servers.

You can now get a basic React app set up following what we've learned in the curriculum (no need to add Redux yet), and you may want to add a `fetch` request into one of your components so that we can later test that our frontend and backend are communicating properly.

### Step 4: Foreman
[Foreman](https://github.com/ddollar/foreman) is a utility that will allow us to run both of our servers at the same time. This way, we can have the React and Rails apps both up on our `localhost` communicating with each other. We could just use two different terminal windows, but Foreman is easy to use and makes the process of starting and stopping your app a lot easier.

First, add Foreman into your `Gemfile`.

```
# Gemfile
gem 'foreman'
```

Then, install.

```$ bundle install```

Create a file called `Procfile` at the root of your Rails app which will hold the processes you'd like Foreman to manage for you. In this file, we will specify how to start the processes for both apps we would like to use. We can name them `web` for our React app, and `api` for the Rails app.

```
# Procfile
web: sh -c 'cd ./client/ && npm start'
api: bundle exec rails s -p 3001
```

Save and close the file, then run:

```$ foreman start -p 3000```

This will tell Foreman to run our React app on port 3000, and since our `Procfile` told it to run the Rails app on port 3001, you should now be able to see your React app running at `localhost:3000`, and your API server at `localhost:3001 `

As [this guide](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/) specifies, you can even create a handy rake task to make starting your servers easy.

```
# lib/tasks/start.rake
task :start do
  exec 'foreman start -p 3000'
end
```

Now startup is as simple as `$ rake start`!

### Step 5: Set up the proxy
There is one last thing to do. In order for your two servers to play nicely together and avoid any Cross-Origin Resource Sharing issues, we need to have Webpack do some fancy proxying magic between them.

Add the following to your `package.json` file, found in your `client` folder:

```
# package.json
"proxy": "http://localhost:3001/",
```

Add `proxy` as top-level key in the file's main object.

You'll want to note that your `fetch` requests don't need to include `localhost:3001` now, due to some configuration that `create-react-app` does and the proxy instructions we just gave to Webpack. For example:

```
# gamesActions.js
export function fetchHighScores() {
  return fetch('/api/v1/games', { method: 'GET'})
    .then(res => res.json())
    .then(games => dispatch({ type: 'ADD_GAMES', payload: games }));
}
```

### Step 6: Make Sure It Works

You should be able to run `$ rake start` now, and your React app should pop up in the browser at `localhost:3000`. You can navigate or make a request from your React app to `localhost:3001` to make sure the backend is working as well. And that should be it for the basic set up! At this point, I started making sure my Rails app could respond with a simple payload of data to a `fetch` request from my React app.

#### A few other tips for the project:
- For the React side of the project, start out by building your components as mostly presentational components. As you begin to realize where you need to maintain state and which components will need to be connected to your Redux state, you can start to transition some components to container components as necessary.
- As a follow on from the above, start building the app just using React state first, and add Redux once you've got the basics working. When you start to find you're duplicating state throughout multiple components, or have state that needs to be access in multiple places, this is a good time to add Redux.
- If you're integrating with a new API, don't forget to use a Web Service Objects to make sure you're separating your concerns appropriately, as we learned [in this lesson](https://learn.co/tracks/full-stack-web-development-v5/rails-and-javascript/consuming-apis/web-service-objects).

#### Helpful Resources
- This site walks you through the same process I've described here and was a really helpful resource for me: [How to get "create-react-app" to work with your Rails API](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/)
- This [fellow student's series](https://emorgan05.github.io/rounding_out_the_api) of blog posts on building a similar project was also very helpful (thanks *emorgan05*!).
