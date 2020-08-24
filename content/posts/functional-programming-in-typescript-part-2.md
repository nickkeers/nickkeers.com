---
title: "Functional Programming in Typescript Part 2"
date: 2020-08-24T22:23:50+01:00
draft: true
---

### Application State

Lets look at the problem with state that I mentioned last time. State is often unavoidable in your programs, its nearly impossible to write programs without state of some form. Even web API's that claim to be "stateless" aren't stateless! You still have state - you need to capture the input data as it comes in, perform validation of some kind, maybe apply some basic transformation to massage the data into an easier to work with format and then finally insert it into a database or perform some form of call out to another service with that data. That's a lot of intermediate steps, so the chances are you end up writing something like this:

```javascript
function myAPIHandler(request) {
    // pull out the data from the request you care about
    const { username, email, password } = parseJSON(request.body)
    let errors = [];

    // check our data and if any of it isn't valid, throw an error
    const usernameCheck = validUsername(username)
    if ( !usernameCheck.valid ) {
        errors.push(usernameCheck.message)
    }

    // as above, omiting for ease of reading
    if ( !validEmail(email) ) {
        // ...
    }
    if ( !validPassword(password) ) {
        // ...
    }

    if ( errors.length > 0 ) {
        // log why our validation failed
        logger.error(errors)

        // return the validation errors back to the frontend to be shown
        return httpBadRequest(errors)
    }

    const record = {
        "username": username,
        "email": email,
        "password": password,
        "timestamp": timestamp()
    }

    try {
        // all was good! inserted our new user into the database
        const insertedRecord = db.insert(record)
        return httpOkResponse()
    } catch (ex) {
        logger.error("Failed to insert record into the database: ", ex.message)
        return serverError()
    }
}
```

There are several places in the above code where you have internal state, and where you have to handle errors - another thing that can be hard to manage, imagine if those were all try-catch's instead. Every time you introduce state into your application, you introduce another area where things can go wrong - you have to protect that state and ensure its not touched by any other part of code and its another piece of information you have to keep in your head. What if you reference the wrong variable further down in your code? You may never notice, especially if you are a solo developer.

