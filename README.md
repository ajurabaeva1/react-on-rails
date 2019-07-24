# Rails as an API!

### Learning Objectives

- Create a REST-ful API in Rails
- Give our API CRUD functionality with ActiveRecord
- Connect our Rails backend with a React frontend
- Make Tess Quotes app with RAILS!!!

# Part One: Making a Rails API

We've talked some about the different flags we can append to `rails new`. Today we'll be using: `--api`.

This does a couple of things:

- Configure your application to start with a more limited set of middleware than normal. Specifically, it will not include any middleware primarily useful for browser applications (like cookies support) by default.
- Make ApplicationController inherit from [ActionController::API](http://api.rubyonrails.org/classes/ActionController/API.html) instead of [ActionController::Base](http://api.rubyonrails.org/classes/ActionController/Base.html). As with middleware, this will leave out any Action Controller modules that provide functionalities primarily used by browser applications.
- Configure the generators to skip generating views, helpers and assets when you generate a new resource.

[(From the docs)](http://edgeguides.rubyonrails.org/api_app.html)

Today, the end goal is a fully functioning Rails app with a React front end. With that in mind, let's get started.

### Generating the Rails app, creating the database, seeding the files

First we need to generate the app and create the database.

```bash
$ rails new tess_rails_api -d postgresql --api -G
$ rails db:create
```

And, as always, it's a good idea to make sure that everything works before we go any farther.

Then, we need to generate the model. For this example, I'm doing a super simple database with just one table. (For your projects, you should have at least two!)

```bash
$ rails g model quote author:string content:text category:string
$ rails db:migrate
```

**Remember**: in Rails, models are singlular. Controllers and routes are plural.

Let's make sure our database works by opening up the rails console and adding a quote.

Lastly, we need to seed the database.

```bash
$ rails db:seed
```

Let's check it out in the Rails console again. Cool? Cool.

### Creating the `index` and `show` methods in the controller

We have to do a couple of things before we can actually get around to sending back JSON data.

First, let's set up our routes. In the end, this will be a fully functioning CRUD app, so let's do this in our `config/routes.rb` file.

```rb
resources :quotes
```

Then, of course, we need to make a controller.

For now, let's just do the `index` and `show` methods -- since we haven't made a frontend, we'll just be visiting URLs and getting JSON.

<details>
<summary>Here's a controller.</summary>

```rb
class QuotesController < ApplicationController
  def index
    @quotes = Quote.all
    render json: { message: "ok", quotes_data: @quotes }
  end

  def show
    begin
      @quote = Quote.find(params[:id])
      render json: { message: "ok", quotes_data: @quote }
    rescue ActiveRecord::RecordNotFound
      render json: { message: "no quote matches that ID" }, status: 404
    rescue Exception
      render json: { message: "there was some other error" }, status: 500
    end
  end

end
```

</details>

### SIDEBAR: Error handling!

You may notice that the `show` method has some error handling built in. It's using a `begin`, `rescue`, (`ensure`), `end` block. Here's how that works:

```rb
begin
  # try to do this thing
rescue NameOfError
  # if there's an error that has this name, do this thing
rescue Exception
  # catch other errors. doing this actually isn't
  # super recommended -- it's better to see exactly what
  # errors you might get and handle them specifically.
ensure
  # anything after ensure will always be done, no matter how
  # many errors happen.
end
```

[Here's a blog post about it.](http://vaidehijoshi.github.io/blog/2015/08/25/unlocking-ruby-keywords-begin-end-ensure-rescue/)

Something similar exists in JavaScript: the `try`/`catch`/`finally` syntax.

```js
try {
  // tryCode - Block of code to try
} catch (err) {
  // catchCode - Block of code to handle errors
} finally {
  // finallyCode - Block of code to be executed
  // regardless of the try / catch result
}
```

[Here's a link to the MDN docs.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)

## 🚀 LAB: Superheroes API!

For this lab, you'll be creating a superheroes API based on some fake data I've provided. For the most part, you'll be able to just follow the steps above, but you'll have to set up the columns in your database table in a particular way for the seed to work.

- `rails g model superhero name:string date_of_birth:string power:string`

Get going!

# Part 2: Hooking Rails up to React!

Since we aren't using views anymore, we need something else to handle the front end interface. What's better than using React? (Nothing. React is the **best**.)

Setting Rails up to work with React is a multi-step process -- similar to setting it up with Express, but there's a couple extra things we need to do, some extra tools, so on and so forth. Most of the tutorials out there have unnecessary steps, or outdated steps, or what have you, so let's walk through this together.

### Creating the React app!

Just like in Express, the React app in a React/Rails setup should be generated with `create-react-app`. (NOTE: There are a couple of gems like `react-rails` and `react-on-rails`. **DO NOT USE THEM. THEY ARE NOT WORTH IT.**)

In the root directory of the Rails app, type `create-react-app client`.

### Running the React app and the Rails app at the same time!

There's a pretty cool gem called [Foreman](https://github.com/ddollar/foreman) that allows us to run both the rails app and the react app at the same time. Here are the steps to set it up:

- `gem install foreman`
- `touch Procfile` (a Procfile is something that allows you to declare multiple processes that should be running for your app. It's actually really interesting, and has to do with the Unix process model. For more information, check out [Process Types and the Procfile](https://devcenter.heroku.com/articles/procfile) and [The Process Model](https://devcenter.heroku.com/articles/process-model)).
- In the Procfile, we have to declare what our two processes are going to be. Add these two lines:

```
web: cd client && npm start
api: bundle exec rails s -p 3001
```

Now, to start the server, the command you'll use is **`foreman start -p 3000`**. (I recommend copy-pasting this command somewhere you'll be able to find it again easily -- I can never remember it.)

Both the Rails server and the React server will start.

### Getting the React app to talk to the Rails app!

In `package.json` in our client directory, we have to make the same change as we did for Express: adding the line `"proxy": "http://localhost:3001",`.

Now we can make fetch requests from the frontend to the backend! Let's do this with the Tess Quotes we've been doing so far.

<details>
<summary>A very simple <code>App.jsx</code></summary>

```
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  constructor() {
    super();
    this.state = {
      apiData: null,
      apiDataLoaded: false,
    };
  }

  componentDidMount() {
    fetch('/quotes').then(res => res.json()).then((jsonRes) => {
      this.setState({
        apiData: jsonRes.quotes_data,
        apiDataLoaded: true,
      });
    });
  }

  showQuotesOnPage() {
    return this.state.apiData.map((quote) => {
      return (
        <div className="quote" key={quote.id}>
          <p>{quote.content}</p>
          <span className="author">{quote.author}</span>
          <span className="category">{quote.category}</span>
        </div>
      );
    });
  }

  render() {
    return (
      <div className="App">
        <div>
          {(this.state.apiDataLoaded) ? this.showQuotesOnPage() : <p>Loading...</p>}
        </div>
      </div>
    );
  }
}

export default App;
```

</details>

## 🚀 LAB: You do with the Superheroes app!

Follow the steps above to create a React frontend for the Superheroes lab. All it needs to do is list the superheroes on the page.

# Part 3: API with CRUD

We learned how to do CRUD with Rails views. Now, let's do it with React. I'll be working from a mostly-completed version of the Tess Quotes app.

> Which methods do Rails controllers have? What ones have we made so far? Which ones do we need to update?

### Create

In order to write the `create` method, we need to do a couple of things:

- format our `fetch` request properly
- check the params to make sure they're correct & what we expect
- add the submitted quote to the database
- send back a message saying the quote has been added to the database & also send back all of the quotes

### Update

The steps for this will be about the same as the ones for create.

- Find quote that needs editing
- Check params
- Update & save quote
- Send back message & all quotes

### Delete

- Find quote by ID
- Delete it
- Send back all quotes

## 🚀 LAB: Fun with the TessQuotes app!!!

For this lab, you'll be working on a version of the Ada Quotes React/Rails app that's missing a couple of features.

- It has the React app... but the `fetch` requests are broken.
- The Rails app exists... but it has no routes or controllers.
- The Rails app and the React app aren't connected.

[Here's a link to the repo.](https://git.generalassemb.ly/wdi-nyc-tesseract/tess_rails_incomplete) Have fun!

## BONUS: Deployment!

Add a script to your React app's `package.json`:

- `"deploy": "cp -a client/build/. public/",`

Then, build the React app:

- From inside the client directory, run this command:
- `npm run build && npm run deploy`

Then, in `config/routes.rb`:

```rb
root to: "root#index"
```

Then, in `controller/root_controller.rb`

```rb
class RootController < ApplicationController
  def index
    render file: "#{Rails.root}/public/index.html"
  end

end
```

Now you can just start the app by running `rails server`, and you're all set up to deploy to Heroku.

Edit your Procfile to say only:

```
web: bundle exec rails server
```

Then, deploy to Heroku:

- `heroku create app_name`
- `git push heroku master`
- `heroku run rails db:migrate`
- `heroku run rails db:seed`
- `heroku open`

Tada!!!
