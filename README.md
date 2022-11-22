# Putting it All Together: Client-Server Communication

## Learning Goals

- Understand how to communicate between client and server using fetch, and how
  the server will process the request based on the URL, HTTP verb, and request
  body
- Debug common problems that occur as part of the request-response cycle

## Introduction

Just like the last lesson, we've got code for a React frontend and Rails API
backend set up. This time though, it's up to you to use your debugging skills to
find and fix the errors!

To get the backend set up, run:

```console
$ bundle install
$ rails db:migrate db:seed
$ rails s
```

Then, in a new terminal, run the frontend:

```console
$ npm install --prefix client
$ npm start --prefix client
```

Confirm both applications are up and running by visiting
[`localhost:4000`](http://localhost:4000) and viewing the list of toys in your
React application.

## Deliverables

In this application, we have the following features:

- Display a list of all the toys
- Add a new toy when the toy form is submitted
- Update the number of likes for a toy
- Donate a toy to Goodwill (and delete it from our database)

The code is in place for all these features on our frontend, but there are some
problems with our API! We're able to display all the toys, but the other three
features are broken.

Use your debugging tools to find and fix these issues.

There are no tests for this lesson, so you'll need to do your debugging in the
browser and using the Rails server logs and `byebug`.

**Note**: You shouldn't need to modify any of the React code to get the
application working. You should only need to change the code for the Rails API.

As you work on debugging these issues, use the space in this README file to take
notes about your debugging process. Being a strong debugger is all about
developing a process, and it's helpful to document your steps as part of
developing your own process.

## Your Notes Here

- Add a new toy when the toy form is submitted

  - How I debugged:
  1. First, I got error **500 (Internal server error)**. This served as a clue to look
  at the server side, the logs especially.

  2. I checked the last error and found "NameError (uninitialized constant ToysController::Toys):".
  With this info, I knew I had to look at the controller folder, and especially the toys_controller.rb file
  to see where the class "Toys" has been used prior to being defined. My focus was especially on the action
  "create", since that is where our post request is routed.

  3. I found the toy object had been created using `toy = Toys.create(toy_params)`. There was an error as
  "Toys" an inexistent class (which is also in line with ActiveRecord's convention of naming classes), was used
  in stead of just "Toy".

  4. I replaced "Toys" with "Toy" in that statement and it worked

- Update the number of likes for a toy

  - How I debugged:
  1. I got the error **Uncaught (in promise) SyntaxError: Unexpected end of JSON input** when I tried liking a toy and 
  knew that whatever I was getting back was not valid JSON.

  2. Since I'm expecting to get back a JSON from the database, I had to check the update method in the controllers (since 
  our request is a patch request and would be routed to it), and see what was being returned

  3. I found the last statement in the update action to be `toy.update(toy_params)`, which is a statement that returns a
  Toy instance object without serializing it to JSON.

  4. I replaced the statement `toy.update(toy_params)` with `render json: toy.update(toy_params)` to make the action returns
  a JSON object that is being expected in the front-end

- Donate a toy to Goodwill (and delete it from our database)

  - How I debugged:
  1. When I tried to "donate a toy to goodwill", I got the error; **404 (Not Found)**. With this error, I knew the reasons
  could be; (1) the object with that id doesn't exist in the database, (2) we are sending the wrong id, (3) we haven't 
  implemented the routes to handle the delete request, or (4) the actual method/action in the controllers is using a different
  id from what is sent to try to delete the object, and the object with that id doesn't exist

  2. Since I knew there was nothing wrong in the front end, I had to check the back-end, starting with checking the routes

  3. I found that even though the destroy action was defined in the controllers, the route that would route delete request
  to it was not defined, so I added it to the list of actions the back-end was expected to handle. The line changed from
  `resources :toys, only: [:index, :create, :update]` to `resources :toys, only: [:index, :create, :update, :destroy]`
