---  
layout: post  
title: "Handling Server-Sent Events with React and Node.js"  
description: "A tutorial showing how to handle events using the Server-Sent Events specification. Create a real-time flight tracker application with React and Node.js."  
date: xxxx-xx-xx xx:xx  
category: Technical Guide, Frontend, Backend, JavaScript  
author:  
  name: "Andrea Chiarelli"  
  url: "https://twitter.com/andychiare"  
  mail: "andrea.chiarelli.ac@gmail.com"  
  avatar: "https://cdn.auth0.com/blog/guest-author/andrea-chiarelli.jpg"  
design:  
  bg_color: "#2f333b"  
  image: https://cdn.auth0.com/blog/xxxxxxxx  
tags:  
- server-sent-events  
- event-source  
- javascript  
- frontend  
- react  
- nodejs  
- backend  
related:  
- xxxxxxxxxx  
- xxxxxxxxxx  
---

**TL;DR:** [Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html) or SSE is a standard enabling a Web server to push data to the client. In this article, we will learn how to use them by building a flight timetable demo application with React and Node.js. However, the concepts you will learn following this tutorial are applicable to any programming language and technology. [You can find the final code of the application in this GitHub repository](https://github.com/andychiare/server-sent-events).

---

## Introducing Server-Sent Events

The typical interaction between a browser and a web server has the browser requesting a resource and the web server providing a response. But how could we describe an event where the server is sending data to the client at any time without an explicit request? If such behavior is to be implemented using the HTTP protocol, it may seem impossible since HTTP only allows the client to request data from the server and not the opposite.

Well, we can do that using our dear friend HTTP. We can use [Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html), also known as _SSE_ or _Event Source_, a W3C standard that allows the server to push data to the client. This may sound like using that annoying polling we'd implement to get the progress status from a long-running server process, but thanks to SSE, we don't have to implement polling to wait for a response from the server. We don't need any complex or strange protocol, we can continue to use the standard HTTP.

So let's take a look at how to use Server-Sent Events in a realistic application.

## Building a Flight Timetable Demo App

In order to show how to use the Server-Sent Events, we are going to develop a simple flight timetable similar to the flight trackers you can find at any airport. The timetable is a simple web page showing a list of flights as shown in the following picture:

![Real-time flights tracker timetable](./xxx-images/flights-timetable.png)

Here, we can find the flight arrival timetable that will automatically update when the status of a flight changes. In our demo application, we are going to simulate the flight status changes using scheduled events; however, this does not affect the realism and scope of our implementation.

Our project is made of two parts:

- A client, where our user interface lives.
- A server, where an endpoint that sends server events to the client resides.

Hence, to better organize our project, let's create a project folder named `flight-timetable`. This folder will act as our project root folder.

So, let's start coding and discover how Server-Sent Events work.

## Starting to Build the React Client

As a first step, let's build our client application. To make things easier, we will use [create-react-app](https://github.com/facebook/create-react-app) to set up the React-based client. So, let's make sure to get installed [Node.js](https://nodejs.org) on our machine and let's type the following command in a shell window:

```shell
npm install -g create-react-app  
```

This command will install `create-react-app` on our computer which will let us create the basic template of a React application.

Ensure that the `flight-timetable` root folder is the current working directory and then type the following command in the shell to scaffold the foundation of our React application:

```shell
create-react-app client  
```

> **Note**: If you have `npm`>5.2 installed, you have `npx` available. `npx` allows you to run `create-react-app` without having to do a global install like so: `npx create-react-app client`. Try it out!

Once this command finishes, under the `flight-timetable` root folder, we will get a `client` folder with everything we need inside. Its `src` subfolder contains the source code of our brand-new application. Our goal is to build our flight timetable application upon this basic React foundation.

```javascript
// client/src/App.js

import React, { Component } from "react";
import logo from "./logo.svg";
import "./App.css";

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```

There are some elements in `client/src/App.js` that we don't need, let's go ahead and clean it up:

```javascript
// client/src/App.js

import React, { Component } from "react";
class App extends Component {
  render() {
    return <div className="App" />;
  }
}

export default App;
```

We should also remove the files that were connected with the lines of code that we just removed. Within the `src` subfolder, let's delete `App.css` and `logo.svg`. You can also remove `App.test.js` if you'd like.

We are going to need to hydrate our application with flight status data as mentioned earlier. Let's create a module that can provide us with such data. Within the `src` folder, let's create a file named `DataProvider.js` and populate it with the following code:

```javascript
// client/src/DataProvider.js

export function getInitialFlightData() {
  return [
    {
      departure: "London",
      flight: "A123",
      arrival: "08:15",
      status: ""
    },
    {
      departure: "Berlin",
      flight: "D654",
      arrival: "08:45",
      status: ""
    },
    {
      departure: "New York",
      flight: "U213",
      arrival: "09:05",
      status: ""
    },
    {
      departure: "Buenos Aires",
      flight: "A987",
      arrival: "09:30",
      status: ""
    },
    {
      departure: "Rome",
      flight: "I768",
      arrival: "10:10",
      status: ""
    },
    {
      departure: "Tokyo",
      flight: "G119",
      arrival: "10:35",
      status: ""
    }
  ];
}
```

This module exports the `getInitialFlightData()` function that returns an array of flight data with each element representing a flight:

```json
{
  "departure": "Rome",
  "flight": "I768",
  "arrival": "10:10",
  "status": ""
}
```

> Notice how each flight starts with a blank `status`.

In a real scenario, the function should request the data from an API, but using the `DataProvider` module is enough for our learning objective.

We now need to provide that data to our `App` component by initializing its state with the flight data that `getInitialFlightData` returns:

```javascript
// client/src/App.js

import React, { Component } from "react";
import { getInitialFlightData } from "./DataProvider";

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      data: getInitialFlightData()
    };
  }
  render() {
    return <div className="App" />;
  }
}

export default App;
```

Since our application will present the flight data in tabular form, we will install a React component that simplifies the task of rendering data in a table: [React Table](https://react-table.js.org). To add this component to our application, we type the following command in the terminal with the `client` folder as our current working directory:

```shell
npm install react-table  
```

Once that's installed, let's import `react-table` into our component and use it to present the data. `react-table` comes with a handy stylesheet that we can also important to add proper styling and structure to our table, saving us a lot of time:

```javascript
// client/src/App.js

import React, { Component } from "react";
import { getInitialFlightData } from "./DataProvider";

import ReactTable from "react-table";
import "react-table/react-table.css";

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      data: getInitialFlightData()
    };
  }
  render() {
    return <div className="App" />;
  }
}

export default App;
```

So, how do we use `ReactTable`? `ReactTable` needs to be passed two important props: `data` and `columns`. As the name may imply, `data` represents any data that we want to show in the table. In our case, `data` would be the flight information we have stored in our `this.state.data`. However, `ReactTable` needs to map the provided data in table columns. To do so, we need to pass it an object that specifies how a provided data object maps into table columns. We can create this definition within our component through a `this.columns` property that maps a property of a flight data object into a `ReactTable` header. Check this out:

```javascript
// client/src/App.js

import React, { Component } from "react";
import { getInitialFlightData } from "./DataProvider";

import ReactTable from "react-table";
import "react-table/react-table.css";

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      data: getInitialFlightData()
    };
    this.columns = [
      {
        Header: "Departure",
        accessor: "departure"
      },
      {
        Header: "Flight",
        accessor: "flight"
      },
      {
        Header: "Arrival",
        accessor: "arrival"
      },
      {
        Header: "Status",
        accessor: "status"
      }
    ];
  }
  render() {
    return (
      <div className="App">
        <ReactTable data={this.state.data} columns={this.columns} />
      </div>
    );
  }
}

export default App;
```

`this.columns` is an array whose elements are objects that define a column of `ReactTable`. Each column object has a `Header` and an `accessor` property. The `Header` is the title that we want to give to that column in our table. The `accessor` is the property name of a data object whose value we want to map to a `Header` column. For example, given the following object, the value of `departure`, which is "Rome", will go under the `Departure` table header, and so on:

```json
{
  "departure": "Rome",
  "flight": "I768",
  "arrival": "10:10",
  "status": ""
}
```

`react-table` has many other options to define a table structure, but `Header` and `accessor` are sufficient for the scope of our flight status application.

With this initial code in place, let's check it out in the browser! To run our app, let's type the following command in the terminal with the `client` folder as the current working directory:

```shell
npm start  
```

The browser should open automatically; if not, please open it and go to <a href="http://localhost:3000" target="_blank">http://localhost:3000</a>. We should see a table of flights as shown in the picture below:

Notice how nicely styled the table is and how we are provided with pagination and row expansion at the bottom. This is all courtesy of `ReactTable`.

## Building the Server

The server-side of our application is a simple Node.js web server responding to requests submitted to the _events_ endpoint. Under the `flight-timetable` root folder, let's create a `server` folder to host all our server related code.

Next, within this folder, let's create a `server.js` file where we create a new instance of [`http.Server`](https://nodejs.org/api/http.html#http_class_http_server) using the Node.js [`http.createServer`](https://nodejs.org/api/http.html#http_http_createserver_options_requestlistener) method:

```javascript
// server/server.js

const http = require("http");
const PORT = 5000;
http.createServer((request, response) => {}).listen(PORT);
console.log(`Server running at http://127.0.0.1:${PORT}/`);
```

To make running our server easier, we are going to install [`nodemon`](https://github.com/remy/nodemon). `nodemon` is a tool that helps us develop Node.js based applications by automatically restarting the Node application when file changes are detected in the directory.

With `server` as our current working directory, let's quickly create a default `package.json` for our Node.js project by running the following command:

```shell
npm init -y  
```

Once this command executes completely, let's install `nodemon` as a developer dependency by running this command:

```shell
npm install --save-dev nodemon  
```

Now, let's open `package.json` and let's change the `start` script so that it uses the `nodemon` command instead of the `node` command:

```json
{
  "name": "server",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "nodemon": "^1.18.2"
  }
}
```

To run the server, let's issue this command in the shell:

```shell
npm start  
```

We'll see the following output in the shell indicating that our server is running and that is under the control of `nodemon`:

```shell
[nodemon] starting `node server.js`  
Server running at http://127.0.0.1:5000/  
```

Now, any time we make a change to `server.js`, our Node application will be automatically restarted.

With a solid development workflow in place, let's build our server further. Within our `createServer` callback, we are going to create business logic that is run whenever a server request to `/events` is made:

```javascript
// server/server.js

const http = require("http");
const PORT = 5000;
http
  .createServer((request, response) => {
    console.log(`Requested URL: ${request.url}`);

    if (request.url.toLowerCase() === `/events`) {
      response.writeHead(200, {
        Connection: "keep-alive",
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache"
      });
    }
  })
  .listen(PORT);
console.log(`Server running at http://127.0.0.1:${PORT}/`);
```

The callback function verifies that the requested URL is `/events` and, only in such case, a response is sent with a few HTTP headers. The headers sent by the server are critical in order to establish a live channel with the client.

The `keep-alive` value of the `Connection` header tells the client to establish a persistent connection: a connection that doesn't end when the first batch of data is received.

The `text/event-stream` value of the `Content-Type` header determines the way the client should interpret the data that it receives. In practice, this value tells the client that we are implementing the Server-Sent Events protocol.

Finally, the `Cache-Control` header asks the client not to store data into its local cache, so that data read by the client comes strictly from the server.

A client can use the [`EventSource API`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) to wait for events coming from the server that report newly available data. We are going to add this to our React application in the next section.

Let test our server connection by issuing a request to our server using the `curl` command in another shell window:

```shell
curl http://127.0.0.1:5000/events  
```

In our server shell, we will see `Requested URL: /events` being output. In the `curl` shell, we see nothing being return yet. Let's now create an event stream to use as reply:

```javascript
// server/server.js

const http = require("http");
const PORT = 5000;
http
  .createServer((request, response) => {
    console.log(`Requested URL: ${request.url}`);

    if (request.url.toLowerCase() === `/events`) {
      response.writeHead(200, {
        Connection: "keep-alive",
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache"
      });

      setTimeout(() => {
        response.write('data: {"flight": "I768", "status": "landing"}');
        response.write("\n\n");
      }, 3000);

      setTimeout(() => {
        response.write('data: {"flight": "I768", "status": "landed"}');
        response.write("\n\n");
      }, 6000);
    }
  })
  .listen(PORT);
console.log(`Server running at http://127.0.0.1:${PORT}/`);
```

We use `setTimeout` to schedule the execution of a few functions in order to simulate a stream of flight `status` change events. This is possible because we never close the server connection. At each scheduled event execution, a string in the following form is sent to the client:

```
data: <DATA>  
```

`<DATA>` represents the data to be sent to the client. In our case, we send a JSON string representing a flight update. We can send multiple data lines in an event response but **the response must be closed by a double empty line**. In other words, our event message could follow this schema to send two data objects:

```
data: This is a message\n  
data: A long message\n  
\n  
\n  
```

Let test our server response again. Issue the `curl` command again:

```shell
curl http://127.0.0.1:5000/events  
```

Observe how, after `6000` ms have elapsed, we have the following output in the `curl` shell:

```shell
data: {"flight": "I768", "status": "landing"}  

data: {"flight": "I768", "status": "landed"}  
```

To complete our server code, let's add logic that responds to requests that do not match `/events`:

```javascript
// server/server.js

const http = require("http");
const PORT = 5000;
http
  .createServer((request, response) => {
    console.log(`Requested URL: ${request.url}`);

    if (request.url.toLowerCase() === `/events`) {
      response.writeHead(200, {
        Connection: "keep-alive",
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache"
      });

      setTimeout(() => {
        response.write('data: {"flight": "I768", "status": "landing"}');
        response.write("\n\n");
      }, 3000);

      setTimeout(() => {
        response.write('data: {"flight": "I768", "status": "landed"}');
        response.write("\n\n");
      }, 6000);
    } else {
      response.writeHead(404);
      response.end();
    }
  })
  .listen(PORT);
console.log(`Server running at http://127.0.0.1:${PORT}/`);
```

If `request.url.toLowerCase()` is not `/events`, we return a `404` error and close the connection. Let's test it out by running this command in the `curl` shell:

```shell
curl http://127.0.0.1:5000/flights  
```

Notice that the shell has no output and immediately ends the connection.

If you want to test this tiny API better, you can use a tool like [Postman](https://www.getpostman.com/) to better visualize the response.

We are now ready to start connecting our client with our server.

## Getting the Server Events

Our client is ready to present data, but right now its sole data provide is the `getInitialFlightData` function that initializes its state. Let's add the functionality it needs to catch server-side generated events.

As we mentioned earlier, we are going to use the [`EventSource`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) API which is the standard interface to [server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events). As MDN documentation says, "_an `EventSource` instance opens a persistent connection to an HTTP server, which sends events in `text/event-stream` format. The connection remains open until closed by calling `EventSource.close()`_", which is exactly what we did in the response headers of our `/events` endpoint in our server.

This, to communicate with the server, we need to add an `eventSource` property to our `App` component and assign it a new `EventSource` instance. It would look something like this:

```javascript
this.eventSource = new EventSource("events");
```

The `EventSource()` constructor creates an object that establishes a communication channel between the client and the server. This channel is unidirectional, meaning that events flow from the server to the client but never in the opposite direction. We pass it as argument a URL that represents an endpoint from where we want to receive events. Since we are using HTTP, the URL can be any valid relative or absolute web address, including query strings.

In our case, to get a stream of events, we need to pass `http://localhost:5000/events` as the argument of `EventSource()`. Let's update our code with this property:

```javascript
// client/src/App.js

// ...

class App extends Component {
  constructor(props) {
    // ...

    this.eventSource = new EventSource("http://localhost:5000/events");
  }

  render() {
    return (
      <div className="App">
        <ReactTable data={this.state.data} columns={this.columns} />
      </div>
    );
  }
}

export default App;
```

However, if we are running our app, we'll notice that in the Developer Console, we have the following error:

```shell
Failed to load http://localhost:5000/events:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
Origin 'http://localhost:3000' is therefore not allowed access.
```

What is happening here?

We want our React client and our Node.js server to communicate with each other; however, they run on different domains: the React client runs on `http://localhost:3000` and the Node.js server runs on `http://localhost:5000`. This means that the JavaScript code of the React client running on your browser cannot make HTTP requests to the Node.js server, due to the [same origin policy](https://en.wikipedia.org/wiki/Same-origin_policy) applied by the browser itself.

We could get around the problem by simply adding a `proxy` value in the `package.json` file, as stated in the [create-react-app documentation](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#proxying-api-requests-in-development). However, due to a [known issue](https://github.com/facebook/create-react-app/issues/3391), currently this workaround is not applicable (If this gets updated, please let me know in the comments at the end of this blog post).

In order to allow the client and server to communicate, we need to enable CORS ([Cross-origin resource sharing](https://auth0.com/docs/cross-origin-authentication)) on our server. With CORS in place, the server authorizes a client hosted on a different domain to request its resources.

To enable CORS in our Node.js server we need to send a new header to the client: the `Access-Control-Allow-Origin` header. Our initial response now becomes as shown below:

```javascript
// server/server.js

// ...

http
  .createServer((request, response) => {
    // ...

    if (request.url.toLowerCase() === `/events`) {
      response.writeHead(200, {
        Connection: "keep-alive",
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache",
        "Access-Control-Allow-Origin": "*"
      });

      // ...
    } else {
      response.writeHead(404);
      response.end();
    }
  })
  .listen(PORT);

console.log(`Server running at http://127.0.0.1:${PORT}/`);
```

The asterisk assigned to the `Access-Control-Allow-Origin` header indicates that any client is authorized to access the resources located at this URL. This, however, may not be the desired solution for a production environment where we should adopt a different approach, such as putting both applications under the same domain, using a reverse proxy, or enabling [CORS](https://auth0.com/docs/cross-origin-authentication) selectively in order to authorize only specific domains.

Let ensure that we save this update to `server.js` so that `nodemon` kicks in. Let's go back to the browser window where our client is live, clear the Developer console, and refresh the page. The `Access-Control-Allow-Origin` error is now gone and our client is finally connected to our server.

Now what we need to do is to capture the events sent by the server. We are going to add an `updateFlightState` method to the `App` component to be called whenever we receive a message from the server and that updates the component state with the message data:

```javascript
// client/src/App.js

// ...

class App extends Component {
  // ...
  updateFlightState(flightState) {
    let newData = this.state.data.map(item => {
      if (item.flight === flightState.flight) {
        item.status = flightState.status;
      }
      return item;
    });

    this.setState(Object.assign({}, { data: newData }));
  }

  render() {
    return (
      <div className="App">
        <ReactTable data={this.state.data} columns={this.columns} />
      </div>
    );
  }
}

export default App;
```

Where should we call `updateFlightState`? We want to start listening for server-sent events once our component is completely settled, thus; the best place to call `updateFlightState` is within `componentDidMount`. What we need to do is to an `EventHandler` that is available with `EventSource` that is called when a `message` event is received, that event handler is `onmessage`:

```javascript
// client/src/App.js

// ...

class App extends Component {
  // ...
  updateFlightState(flightState) {
    let newData = this.state.data.map(item => {
      if (item.flight === flightState.flight) {
        item.status = flightState.status;
      }
      return item;
    });

    this.setState(Object.assign({}, { data: newData }));
  }

  componentDidMount() {
    this.eventSource.onmessage = e =>
      this.updateFlightState(JSON.parse(e.data));
  }

  render() {
    return (
      <div className="App">
        <ReactTable data={this.state.data} columns={this.columns} />
      </div>
    );
  }
}

export default App;
```

In `componentDidMount`, the `onmessage` property of `this.eventSource` stores an event handler that is called when an event arrives from the server. In our case, the assigned event handler calls `updateFlightState()` to update the component state with the new data sent by the server. Each server message event carries data in the `e.data` property represented as a JSON string; therefore, we need to parse that string into a JSON object that then we can use to find and update the corresponding flight within the body of `updateFlightState`.

Now everything is ready to update our flight table in real time! Refresh the client application. After a few seconds, we should see our browser showing something like the following:

![Real-time flights app timetable example](./xxx-images/animated-flights-timetable.gif)

Flight I768 goes from no status to "landing" status and, finally, to "landed" status.

## Managing Event Types

The timetable application developed so far responds to server events always in the same way: by updating the specific flight sent in the event `data` property.

But suppose, for example, that we want to remove the row describing a flight `X seconds` after it has landed. How could the server transmit an event that is not related to a flight status change? How can the client capture such event and remove the respective row from the table?

We certainly need to introduce some way to differentiate our server-sent events. We need to provide them with types.

We could think of using the `data` property of the event to specify a differentiating characteristic of the event being sent. However, the Server-Sent Events protocol let us specify an `event` property as part of our message that we can use to easily transmit the event type to the client.

We can then create two types of events, `flightStateUpdate` and `flightRemoval` by sending an `event` property with our response, just like this:

```javascript
setTimeout(() => {
  response.write("event: flightStateUpdate\n");
  response.write('data: {"flight": "I768", "status": "landed"}');
  response.write("\n\n");
}, 6000);

setTimeout(() => {
  response.write("event: flightRemoval\n");
  response.write('data: {"flight": "I768"}');
  response.write("\n\n");
}, 9000);
```

It's important that we send the `event` message first so that the client knows what to do with the data it receives much easier. We can think about it as a `command => action` flow.

> Client: First, tell me the type of event I will be handling so that I know what to do once I received the data.

Let's integrate this event-typing in our `server.js` code:

```javascript
// server/server.js

// ...

http
  .createServer((request, response) => {
    // ...

    if (request.url.toLowerCase() === `/events`) {
      // ...

      setTimeout(() => {
        response.write("event: flightStateUpdate\n");
        response.write('data: {"flight": "I768", "status": "landing"}');
        response.write("\n\n");
      }, 3000);
      setTimeout(() => {
        response.write("event: flightStateUpdate\n");
        response.write('data: {"flight": "I768", "status": "landed"}');
        response.write("\n\n");
      }, 6000);
      setTimeout(() => {
        response.write("event: flightRemoval\n");
        response.write('data: {"flight": "I768"}');
        response.write("\n\n");
      }, 9000);
    } else {
      response.writeHead(404);
      response.end();
    }
  })
  .listen(PORT);

console.log(`Server running at http://127.0.0.1:${PORT}/`);
```

If we refresh the browser window where the client is live, we will not see any changes happening in the table. That's because we have not adapted our client code to handle event-typing.

The client has to handle these typed events by creating custom event listeners on `this.eventSource` using the `addEventListener` method. For example, to listen to the `flightStateUpdate` server event, we would need to create an event listener like this:

```javascript
this.eventSource.addEventListener("flightStateUpdate", e =>
  this.updateFlightState(JSON.parse(e.data))
);
```

This is identical to creating regular `DOM` event listeners, but here, we use our custom event names. We can do the same for a `flightRemoval` event or any other event we may need to create.

Let's update our code to listen to our custom server-sent events:

```javascript
// client/src/App.js

// ...

class App extends Component {
  constructor(props) {
    // ...
    this.eventSource = new EventSource("http://localhost:5000/events");
  }

  updateFlightState(flightState) {
    let newData = this.state.data.map(item => {
      if (item.flight === flightState.flight) {
        item.status = flightState.status;
      }
      return item;
    });

    this.setState(Object.assign({}, { data: newData }));
  }

  removeFlight(flightInfo) {
    const newData = this.state.data.filter(
      item => item.flight !== flightInfo.flight
    );

    this.setState(Object.assign({}, { data: newData }));
  }

  componentDidMount() {
    this.eventSource.addEventListener("flightStateUpdate", e =>
      this.updateFlightState(JSON.parse(e.data))
    );
    this.eventSource.addEventListener("flightRemoval", e =>
      this.removeFlight(JSON.parse(e.data))
    );
  }

  render() {
    return (
      <div className="App">
        <ReactTable data={this.state.data} columns={this.columns} />
      </div>
    );
  }
}

export default App;
```

The `componentDidMount` method no longer assigns an event handler to the `onmessage` property of `this.eventSource`. Now we are using the `addEventListener()` method in order to assign an event handler to a specific event. We are able to easily assign a specific event handler to each event generated by the server as if it was generated by any standard HTML element.

We assigned the `updateFlightState()` method to the `flightStateUpdate` event. We also created a `removeFlight()` method that uses the JavaScript array `filter` operator to remove a flight. `removeFlight()` is assigned to the `flightRemoval` event.

Refresh the browser and witness not only the status of flight I768 changing, but its row also being removed a few seconds after it has landed.

## Handling Connection Closure

The Server-Sent Event connection between the client and the server is a streaming connection. This means that the connection will be kept active indefinitely unless the client or the server stops running. If our server has no more events to send or the client isn't longer interested in server's events, how can we explicitly stop the currently active connection?

Let's consider the case where the client wants to stop the event stream. Let's create a button that allows the user to stop receiving new events. So, let's change the `render()` method of the `App` class, as shown below:

````react
 render() { return ( <div className="App"> <button onClick={() => this.stopUpdates()}>Stop updates</button> <ReactTable data={this.state.data} columns={this.columns} /> </div> ); }```  

Here we added a `<button>` element and bound the click event to the `stopUpdates()` method of the same component. This method will look like the following:  

```javascript  
 stopUpdates() { this.eventSource.close();  
 }```  

In other words, to stop the event stream we simply invoked the `close()` method of the `eventSource` object.  

Closing the event stream on the client doesn't automatically closes the connection on the server side. This means that the server will continue to send events to the client. We need to intercept on the server side the request of connection closing. We can do it by adding an event handler for the `close` event on the server side, as shown by the following code:  

```javascript  
// server.js  

const http = require("http");  
http  
 .createServer((request, response) => {  
  console.log(`Request url: ${request.url}`);  

 const eventHistory = [];  
  request.on("close", () => {  
  closeConnection(response);  
 });  
 if (request.url.toLowerCase() === "/events") {  
  response.writeHead(200, {  
  Connection: "keep-alive",  
 "Content-Type": "text/event-stream", "Cache-Control": "no-cache", "Access-Control-Allow-Origin": "*" });  
 //Event generation code //.. } else {  response.writeHead(404);  
  response.end();  
 } }) .listen(5000, () => {  console.log("Server running at http://127.0.0.1:5000/");  
 });  
function closeConnection(response) {  
 if (!response.finished) {  
  response.end();  
  console.log("Stopped sending events.");  
 }}  
````

The `request.on()` method catches the `close` request and executes the `closeConnection()` function. The `closeConnection()` method invokes the `response.end()` method to close the HTTP connection. It also checks if the connection is already closed: this situation may occur when multiple closing requests are sent by the client.

Since our server event generators are scheduled by using `setTimeout()`, it could happen that an attempt to send an event to the client may be made after the connection has been closed, raising an exception. We can avoid this by checking if the connection is still active, as shown in the following example:

```javascript
setTimeout(() => {
  if (!response.finished) {
    response.write("event: flightStateUpdate\n");
    response.write('data: {"flight": "I768", "state": "landing"}');
    response.write("\n\n");
  }
}, 3000);
```

When the connection closure has to be originated by the server, it will be done by invoking the `response.end()` method, as seen before. Also, the client should be informed about the closure, so that it can free resources on its side. What usually happens is the generation of a specific event that delegates to the client to request the connection closing. For example, our server could simply generate a `closedConnection` event as follows:

```javascript
setTimeout(() => {
  if (!response.finished) {
    response.write("event: closedConnection\n");
    response.write("data: ");
    response.write("\n\n");
  }
}, 3000);
```

This event should be handled by the client as in the following:

```react
// src/App.js  

import React, { Component } from 'react';  
import ReactTable from 'react-table';  
import { getInitialFlightData } from './DataProvider';  
import 'react-table/react-table.css';  

class App extends Component {  
 constructor(props) { //Initialization code }  
 componentDidMount() { this.eventSource.addEventListener('flightStateUpdate', (e) => this.updateFlightState(JSON.parse(e.data))); this.eventSource.addEventListener('flightRemoval', (e) => this.removeFlight(JSON.parse(e.data))); this.eventSource.addEventListener('closedConnection', (e) => this.stopUpdates()); }  
 updateFlightState(flightState) { //Update state code }  
 removeFlight(flightInfo) { //Remove flight code }  
 stopUpdates() { this.eventSource.close(); }  
 render() { //Render code }}  

export default App;  
```

It assigns the `stopUpdates()` method to the `closedConnection` event and starts the connection closure process we already saw above.

## Handling Connection Recovery

So far we built a complete event system management: we are able to get different types of events pushed by the server and to control the end of the event stream. But what happens if the client loses some event due to network issues? Of course, it depends on the specific application. In some situations, we may ignore the loss of some event, in some others we can't.

Let's consider, for example, the event stream we implemented so far. If a network issue happens and the client loses the `flightStateUpdate` event that puts the flight into the landing state, it could not be a big problem. Simply the user will not be able to see the landing phase on the timetable, but when the connection will be restored the timetable will provide correct information with the subsequent states. However, if the network issue happens immediately after the flight enters in the landing state and the connection is restored after the `flightRemoval` event, we have an issue: the flight will remain in the landing state forever and we need to handle this bad situation.

The Server-Sent Events protocol help us by providing a mechanism to identify events and to restore a dropped connection. Let's try to explain.

When the server generates an event, we have the ability to assign an identifier by attaching an `id` keyword to the response to be sent to the client. For example, we could send the `flightStateUpdate` event as shown by the following code:

```javascript
setTimeout(() => {
  if (!response.finished) {
    const eventString =
      'id: 1\nevent: flightStateUpdate\ndata: {"flight": "I768", "state": "landing"}\n\n';
    response.write(eventString);
  }
}, 3000);
```

Here we did a little refactoring of the code and added the `id` keyword with 1 as its value.

When a network issue happens and the connection to an event stream is lost, the browser will automatically attempt to restore the connection. When the connection is established again, the browser will automatically send the identifier of the last received event in the `Last-Event-Id` HTTP header. So, the server should be changed in order to handle this request of restoring the event stream. It is up to the server to decide if sending all missed events or continue with newly generated events. Anyway, if the server needs to send all missed events, it also needs to store all events already sent to the client.

Let's implement this strategy in our server. With a bit of refactoring, the following is the final version of the server side code:

```javascript
// server.js

const http = require("http");
http
  .createServer((request, response) => {
    console.log(`Request url: ${request.url}`);

    const eventHistory = [];
    request.on("close", () => {
      closeConnection(response);
    });
    if (request.url.toLowerCase() === "/events") {
      response.writeHead(200, {
        Connection: "keep-alive",
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache",
        "Access-Control-Allow-Origin": "*"
      });
      checkConnectionToRestore(request, response, eventHistory);

      sendEvents(response, eventHistory);
    } else {
      response.writeHead(404);
      response.end();
    }
  })
  .listen(5000, () => {
    console.log("Server running at http://127.0.0.1:5000/");
  });
function sendEvents(response, eventHistory) {
  setTimeout(() => {
    if (!response.finished) {
      const eventString =
        'id: 1\nevent: flightStateUpdate\ndata: {"flight": "I768", "state": "landing"}\n\n';
      response.write(eventString);
      eventHistory.push(eventString);
    }
  }, 3000);
  setTimeout(() => {
    if (!response.finished) {
      const eventString =
        'id: 2\nevent: flightStateUpdate\ndata: {"flight": "I768", "state": "landed"}\n\n';
      response.write(eventString);
      eventHistory.push(eventString);
    }
  }, 6000);
  setTimeout(() => {
    if (!response.finished) {
      const eventString =
        'id: 3\nevent: flightRemoval\ndata: {"flight": "I768"}\n\n';
      response.write(eventString);
      eventHistory.push(eventString);
    }
  }, 9000);
  setTimeout(() => {
    if (!response.finished) {
      const eventString = "id: 4\nevent: closedConnection\ndata: \n\n";
      eventHistory.push(eventString);
    }
  }, 12000);
}

function closeConnection(response) {
  if (!response.finished) {
    response.end();
    console.log("Stopped sending events.");
  }
}

function checkConnectionToRestore(request, response, eventHistory) {
  if (request.headers["last-event-id"]) {
    const eventId = parseInt(request.headers["last-event-id"]);

    eventsToReSend = eventHistory.filter(e => e.id > eventId);

    eventsToReSend.forEach(e => {
      if (!response.finished) {
        response.write(e);
      }
    });
  }
}
```

We introduced the `eventHistory` array, where the events sent to the client are stored.

Then we assigned to the `checkConnectionToRestore()` function the task of checking and restoring possible broken connections and encapsulated the code for event generation in the `sendEvents()` function.

We can see that now each time an event is sent to the client, it is also stored in the `eventHistory` array, as we can check in the following code:

```javascript
setTimeout(() => {
  if (!response.finished) {
    const eventString =
      'id: 1\nevent: flightStateUpdate\ndata: {"flight": "I768", "state": "landing"}\n\n';
    response.write(eventString);
    eventHistory.push(eventString);
  }
}, 3000);
```

If the `checkConnectionToRestore()` function find the `Last-Event-Id` HTTP header in the request, it filters the already sent events in the `eventHistory` array and sends them again to the client, as we can see in the code below:

````javascript
function checkConnectionToRestore(request, response, eventHistory) {  
 if (request.headers['last-event-id']) {  
 const eventId = parseInt(request.headers['last-event-id']);  

  eventsToReSend = eventHistory.filter((e) => e.id > eventId);  

  eventsToReSend.forEach((e) => {  
 if (!response.finished) {  
  response.write(e);  
 } }); }```  

This completes and makes more robust our system.  

## Browser Support  

According to [caniuse.com](https://caniuse.com/#search=server%20sent%20events), Server-Sent Events are currently supported by all major browsers but Internet Explorer, Edge and Opera Mini. Although [supporting them in Edge is under consideration](https://developer.microsoft.com/en-us/microsoft-edge/platform/status/serversenteventseventsource/), the lack of universal support forces us to use polyfills, such as [Remy Sharp's EventSource.js](https://github.com/remy/polyfills/blob/master/EventSource.js) or [Yaffle's EventSource](https://github.com/Yaffle/EventSource) or [AmvTek's EventSource](https://github.com/amvtek/EventSource).  

Using these polyfills is very simple. Here is an example of using AmvTek's polyfill, but using the other ones is not so different. In order to add AmvTek's EventSource polyfill to our React client application, we need to install it via `npm`, as shown below:  

```shell  
npm install eventsource-polyfill  
````

Now we should import the module in the `App` component's module:

```react
// src/App.js  

import React, { Component } from 'react';  
import ReactTable from 'react-table';  
import 'react-table/react-table.css';  
import { getInitialFlightData } from './DataProvider';  
import 'eventsource-polyfill'  

class App extends Component {  
 //Component code}  
```

That's all. The polyfill will define an `EventSource` constructor only if it is not natively supported and our code will continue to work as before also on browsers that don't support it.

## Server-Sent Events vs WebSockets

Using Server Side Events helps us to resolve some common issues that involve waiting for data from the server. Instead of implementing a long polling whose main drawback is consuming resources both on the client and on the server side, we get a simple and performant solution.

An alternative approach is based on [WebSockets](https://www.w3.org/TR/websockets/), a standard TCP based protocol providing full duplex communication between the client and the server. What benefits does WebSockets bring with respect to Server-Sent Events? When to use one technology instead of the other?

The following are some considerations to keep in mind when choosing between Server-Sent Events and WebSockets:

- WebSockets supports a bidirectional communication, while Server-Sent Events supports only communication from the server to the client
- WebSockets is a low-level protocol, while Server-Sent Events is based on HTTP and so it doesn't require additional settings in the network infrastructure
- WebSockets supports binary data transfer, while Server-Sent Events supports only text-based data transfer; if we want to transfer binary data via Server-Sent Events we need to encode it in Base64
- The combination of low-level protocol and support of binary data transfer makes WebSockets more suitable than Server-Sent Events for applications requiring real-time binary data transfer as may happen in gaming or other similar application types

## Summary

In this article, we used Server-Sent Events to implement a client-server application simulating a flight timetable. During the implementation, we had the opportunity to explore the features that the standard provides us in order to support event typing, connection control and restoring. We also faced the problem to add polyfills for browsers not supporting Server-Sent Events and we showed a comparison with WebSockets.

You can download the final code of the project developed throughout the post from [this GitHub repository](https://github.com/andychiare/server-sent-events).
