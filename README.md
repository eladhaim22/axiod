# Axiod

Promise based HTTP client for Deno inpired by axios

## Features

- Make http requests from node.js
- Supports the Promise API
- Intercept request and response
- Transform request and response data
- Automatic transforms for JSON data
- Cancel requests [Waiting for deno support of Fetch Abort using signal and AbortController]

## Usage

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod.get("https://google.fr").then((response) => {
  // response
});
```

Performing a `GET` request

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod
  .get("/user?ID=12345")
  .then( (response) => {
    // handle success
    console.log(response);
  })
  .catch( (error) => {
    // handle error
    console.log(error);
  })
  .then( () => {
    // always executed
  });

// Optionally the request above could also be done as
axiod
  .get("/user", {
    params: {
      ID: 12345,
    },
  })
  .then( (response) => {
    console.log(response);
  })
  .catch( (error) => {
    console.log(error);
  })
  .then( () => {
    // always executed
  });

// Want to use async/await? Add the `async` keyword to your outer function/method.
const getUser = () => {
  try {
    const response = await axiod.get("/user?ID=12345");
    console.log(response);
  } catch (error) {
    console.error(error);
  }
}
```

Performing a `POST` request

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod
  .post("/user", {
    firstName: "Fred",
    lastName: "Flintstone",
  })
  .then((response) => {
    console.log(response);
  })
  .catch((error) => {
    console.log(error);
  });
```

Performing multiple concurrent requests

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

const getUserAccount = () => {
  return axiod.get("/user/12345");
};

const getUserPermissions = () => {
  return axiod.get("/user/12345/permissions");
};

Promise.all([getUserAccount(), getUserPermissions()]).then((results) => {
  const acct = results[0];
  const perm = results[1];
});
```

## Axiod API

Requests can be made by passing the relevant config to axiod.

**axiod(config)**
// Send a POST request

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod({
  method: "post",
  url: "/user/12345",
  data: {
    firstName: "Fred",
    lastName: "Flintstone",
  },
});
```

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

// GET request for remote image in node.js
axiod({
  method: "get",
  url: "http://bit.ly/2mTM3nY",
  responseType: "stream",
}).then((response) => {
  response.data.pipe(fs.createWriteStream("ada_lovelace.jpg"));
});
```

**axiod(url[, config])**
// Send a GET request (default method)

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod("/user/12345");
```

### Request method aliases

For convenience aliases have been provided for all supported request methods.

```
axiod.request(config)
axiod.get(url[, config])
axiod.delete(url[, config])
axiod.head(url[, config])
axiod.options(url[, config])
axiod.post(url[, data[, config]])
axiod.put(url[, data[, config]])
axiod.patch(url[, data[, config]])
```

**NOTE**

> When using the alias methods url, method, and data properties don't need to be specified in config.

### Creating an instance

You can create a new instance of axiod with a custom config.

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

// axiod.create([config]);
const instance = axiod.create({
  baseURL: "https://some-domain.com/api/",
  timeout: 1000,
  headers: { "X-Custom-Header": "foobar" },
});
```

## Request Config

These are the available config options for making requests. Only the url is required. Requests will default to GET if method is not specified.

```javascript
{
  // `url` is the server URL that will be used for the request
  url: '/user',

  // `method` is the request method to be used when making the request
  method: 'get', // default

  // `baseURL` will be prepended to `url` unless `url` is absolute.
  // It can be convenient to set `baseURL` for an instance of axiod to pass relative URLs
  // to methods of that instance.
  baseURL: 'https://some-domain.com/api/',

  // `headers` are custom headers to be sent
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` are the URL parameters to be sent with the request
  // Must be a plain object or a URLSearchParams object
  params: {
    ID: 12345
  },

  // `paramsSerializer` is an optional function in charge of serializing `params`
  paramsSerializer: function (params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` is the data to be sent as the request body
  // Only applicable for request methods 'PUT', 'POST', 'DELETE , and 'PATCH'
  // When no `transformRequest` is set, must be of one of the following types:
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - Browser only: FormData, File, Blob
  // - Node only: Stream, Buffer
  data: {
    firstName: 'Fred'
  },

  // syntax alternative to send data into the body
  // method post
  // only the value is sent, not the key
  data: 'Country=Brasil&City=Belo Horizonte',

  // `timeout` specifies the number of milliseconds before the request times out.
  // If the request takes longer than `timeout`, the request will be aborted.
  timeout: 1000, // default is `0` (no timeout)

  // `withCredentials` indicates whether or not cross-site Access-Control requests
  // should be made using credentials
  withCredentials: false, // default

  // `auth` indicates that HTTP Basic auth should be used, and supplies credentials.
  // This will set an `Authorization` header, overwriting any existing
  // `Authorization` custom headers you have set using `headers`.
  // Please note that only HTTP Basic auth is configurable through this parameter.
  // For Bearer tokens and such, use `Authorization` custom headers instead.
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  }

  // `validateStatus` defines whether to resolve or reject the promise for a given
  // HTTP response status code. If `validateStatus` returns `true` (or is set to `null`
  // or `undefined`), the promise will be resolved; otherwise, the promise will be
  // rejected.
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },


  // `transformRequest` allows changes to the request data before it is sent to the server
  // This is only applicable for request methods 'PUT', 'POST', 'PATCH' and 'DELETE'
  // The last function in the array must return a string or an instance of Buffer, ArrayBuffer,
  // FormData or Stream
  // You may modify the headers object.
  transformRequest: [function (data, headers) {
    // Do whatever you want to transform the data

    return data;
  }],

  // `transformResponse` allows changes to the response data to be made before
  // it is passed to then/catch
  transformResponse: [function (data) {
    // Do whatever you want to transform the data

    return data;
  }],
}
```

## Response Schema

The response for a request contains the following information.

```javascript
{
  // `data` is the response that was provided by the server
  data: {},

  // `status` is the HTTP status code from the server response
  status: 200,

  // `statusText` is the HTTP status message from the server response
  statusText: 'OK',

  // `headers` the HTTP headers that the server responded with
  // All header names are lower cased and can be accessed using the bracket notation.
  // Example: `response.headers['content-type']`
  headers: {},

  // `config` is the config that was provided to `axiod` for the request
  config: {}
}
```

When using then, you will receive the response as follows:

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod.get("/user/12345").then((response) => {
  console.log(response.data);
  console.log(response.status);
  console.log(response.statusText);
  console.log(response.headers);
  console.log(response.config);
});
```

## Handling Errors

```javascript
import axiod from "https://deno.land/x/axiod/mod.ts";

axiod.get("/user/12345").catch((error) => {
  if (error.response) {
    // The request was made and the server responded with a status code
    // that falls out of the range of 2xx
    console.log(error.response.data);
    console.log(error.response.status);
    console.log(error.response.headers);
  } else {
    // Something happened in setting up the request that triggered an Error
    console.log("Error", error.message);
  }
  console.log(error.config);
});
```

## License

MIT
