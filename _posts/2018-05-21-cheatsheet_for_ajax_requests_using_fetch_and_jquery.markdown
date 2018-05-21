---
layout: post
title:      "Cheatsheet for AJAX requests using Fetch & JQuery"
date:       2018-05-21 15:26:50 +0000
permalink:  cheatsheet_for_ajax_requests_using_fetch_and_jquery
---


When working on adding AJAX features to my Rails app, I came across a few different ways of making AJAX requests to my Rails backend. Keeping the various methods and their syntax straight was confusing so I decided to make this cheat sheet:

### Fetch
```
fetch("http://localhost:3000/patients")
  .then(function (response) {
      response.json().then(doSomethingWithTheJSON)
    })
  .catch (errorFunction)
```

`fetch()` returns a Promise, and calling `json()` on the response from `fetch()` also returns a Promise, so you are then free to chain whatever callbacks you need to handle the response data.

### JQuery Methods

****$.ajax()**** - lower level

```
$.ajax({
    method: "GET",
    url: "/patients",
    dataType: "json"
})
.done(successFunction)
.error(errorFunction);
```

****$.get()**** - shorthand for $.ajax()
```
$.get( "url", successFunction).fail(errorFunction);
```

****$.post()**** - shorthand for $.ajax()
```
$.post( "url", successFunction).fail(errorFunction);
```

All of the JQuery methods return "superset of the XMLHTTPRequest object." JQuery calls these objects jqXHR objects, and they all implement the Promise interface, so you can chain callbacks just like you would with any other method returning a Promise.

#### Summing Up
See the below resources for more information about Promises and how they're used:
  - [JavaScript Promises: an Introduction](https://developers.google.com/web/fundamentals/primers/promises)
  - [MDN Promise Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
  - [MDN json() Reference ](https://developer.mozilla.org/en-US/docs/Web/API/Body/json)
