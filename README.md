# hapi-pagination

**Forked from the original hapi-pagination at https://github.com/fknop/hapi-pagination**

Hapi plugin to handle 'custom' resources pagination in json only.
Version 5.0.0 is updated for use with @hapi/hapi 21.3.3 and later, and updates all dependencies to the latest compatible versions to resolve vulnerabilities.

## How to install

```
npm install @userfront/hapi-pagination --save
```

## Contribute

Post an issue if you encounter a bug or an error in the documentation.
Create a new pull request if you want to add a new functionality or do any change that might get the plugin better.

## CHANGELOG

For versions prior to 5.0.0, check the original package's release log: https://github.com/fknop/hapi-pagination/releases
For versions 5.0.0 and later, check CHANGELOG.md

## How to use

Note: If you're reading this on npm, the README may not be up to date.

The plugin works with settings that you can override. You can override the
default value of a setting and even the default name. It allows you to customize
your calls to your API to suit your needs.

### Options

See the default options object below.

#### Root-level parameters

* `zeroIndex`: Defaults to `false`; change this to `true` if you want to start paginating at page 0 instead of page 1

#### The query parameters

The plugin accepts query parameters to handle the pagination, you can customize
these parameters with the following options:

* `limit`: The number of resources by page. Default value is 25, default name is
  limit.
* `page`: The number of the page that will be returned. Default value is 1,
  default name is page.
* `pagination`: Allows you to enable, disable pagination for one request. Default
  value is true (enabled), default name is pagination, enabled by default.
* `invalid`: This is `NOT` a query parameter, but it allows you to customize the
  behavior if the validation of limit and page fails. By default, it sets the
  defaults, you can set it to 'badRequest' that will send you a `400 - Bad
  Request`.

Notes:
* You can access to limit, page and pagination in the handler method through `request.query`.
* If the pagination is set to false, the metadata object will not be a part of
  the response but the `pagination` parameter will still be accessible through
  `request.query`

#### The metadata

By default the plugin will generate a metadata object alongside your resources in the response body.
You can also decide to put the metadata in the response header, so the body remains clean.
In this case, the plugin will use 2 kinds of headers:
* `Content-Range`: `startIndex-endIndex/totalCount` (see https://tools.ietf.org/html/rfc7233#section-4.2)
  This header gives the start and end indexes of the current page records, followed by the total number of records.  In the case that `meta.alwaysIncludeHeaders` is `true` and the result set is empty the value will instead take the form `*/totalCount` per RFC 7233.
* `Link`: `<url>; rel=relationship` (see https://tools.ietf.org/html/rfc2068#section-19.6.2.4)
  This header gives the url to different resources related to the current page. The available relationships are :
    + `rel=self`: the current page
    + `rel=first`: the first page
    + `rel=prev`: the previous page
    + `rel=next`: the next page
    + `rel=last`: the last page

To choose where the metadata will be put, use the option meta.location (see below).
You can customize the metadata with the following options:


* `name`: The name of the metadata object. Default is 'meta'.
* `baseUri`: The base uri for the generated links. Default is undefined. Relative URI can be achieved with an empty string
* `count`: The number of rows returned. Default name is count. Enabled by default.
* `totalCount`: The total numbers of rows available. Default name is totalCount.
  Enabled by default.
* `pageCount`: The total number of pages available. Default name is pageCount,
  enabled by default.
* `self`: The link to the requested page. Default name is self, enabled by
  default.
* `previous`: The link to the previous page. Default name is previous, enabled by
  default. null if no previous page is available.
* `next`: Same than previous but with next page.
* `hasNext`: A boolean indicating there is a next page. Disabled by default
* `hasPrevious`: A boolean indicating there is a previous page. Disabled by default
* `first`: Same than previous but with first page.
* `last`: Same than previous but with last page.
* `page`: The page number requested. Default name is page, disabled by default.
* `limit`: The limit requested. Default name is limit, disabled by default.
* `location`: 'body' put the metadata in the response body, 'header' put the metadata in the response header. Default is 'body'.
* `alwaysIncludeHeaders`: when `location` is 'header' and this option is set to `true`, the Content-Range and Link headers will be set even when the result set is empty, and when the entire result set fits on a single page.
* `rangeNotSatisfiableStatusCode`: when `alwaysIncludeHeaders` is `true` and the requested page is out of range of the result set, this status code will be used.
* `successStatusCode`: HTTP response status code when returning paginated data, undefined by default so the code set by the application prevails.

#### The results

* `name`: the name of the results array, results by default. Useful with location='body' only.
* `reply`: Object with:
    + `paginate`: The name of the paginate method (see below), paginate by
    default.

#### The routes

* `include`: An array of routes that you want to include, support \* and regex.
  Default to '\*'.
* `exclude`: An array of routes that you want to exclude. Useful when include is
  '\*'. Default to empty array. Support regex.

#### Override on route level

You can override the page, limit, and pagination default value on route level.
You can also force enable or disable the pagination on a route level. This is
useful when you're using regex for example.


```javascript
config: {
  plugins: {
    pagination: {
      // enabled: boolean - force enable or force disable
      defaults: {
        // page: override page
        // limit: override limit
        // pagination: override if pagination is false or true by
        // default
      }
    }
  }
}
```

#### h.paginate(Array|Object, [totalCount], [options = {}])

The method is an helper method. This is a shortcut for:

```javascript
h.response({results: results, totalCount: totalCount});
```

You can change names of fields (`results`, `totalCount`) using reply options.

```
reply: {
  results: {
    name: 'rows'
  },
  totalCount: {
    name: 'count'
  }
}
```

You can also reply the array and set the totalCount by adding the totalCount
(with whatever name you chose) to the request object.

```
request.totalCount = 10;
h.response(results);
```

The `paginate` method also offers a way to add custom properties to your response. You just have to
pass an object as first parameter and pass a `options.key` parameter which is the name of the key of the paginated results.

For example:

```
return h.paginate({ results: [], otherKey: 'value', otherKey2: 'value2' }, 0, { key: 'results' });
```

The response will also contains `otherKey` and `otherKey2`. Nested keys for the paginated results are not allowed.

If you pass an object but forgot to pass a key for your results, the paginate method will throw an error. Same thing if the key does not exist.

Please note that if you pass metadata in headers the custom properties don't work, because we don't want to change the response in this case.

##### WARNING: If the results is not an array, the program will throw an implementation error.

If totalCount is not exposed through the request object
or the h.paginate method, the following attributes will be
set to null if they are active.
 * `last`
 * `pageCount`
 * `totalCount`
 * `next`

You can still have those four attributes by exposing totalCount even if
totalCount is set to false.

#### The defaults options

```javascript
const options = {
    query: {
        page: {
            name: 'page',
            default: 1
        },
        limit: {
            name: 'limit',
            default: 25
        },
        pagination: {
            name: 'pagination',
            default: true,
            active: true
    }
        invalid: 'defaults'
    },

    meta: {
        location: 'body',
        successStatusCode: undefined,
        name: 'meta',
        count: {
            active: true,
            name: 'count'
        },
        totalCount: {
            active: true,
            name: 'totalCount'
        },
        pageCount: {
            active: true,
            name: 'pageCount'
        },
        self: {
            active: true,
            name: 'self'
        },
        previous: {
            active: true,
            name: 'previous'
        },
        next: {
            active: true,
            name: 'next'
        },
        hasNext: {
            active: false,
            name: 'hasNext'
        },
        hasPrevious: {
            active: false,
            name: 'hasPrevious'
        },
        first: {
            active: true,
            name: 'first'
        },
        last: {
            active: true,
            name: 'last'
        },
        page: {
            active: false,
            // name == default.query.page.name
        },
        limit: {
            active: false
            // name == default.query.limit.name
        }
    },

    results: {
      name: 'results'
    },
    reply: {
        paginate: 'paginate',
        results: {
          name: 'results'
        },
        totalCount:{
          name: 'totalCount'
        }
    },

    routes: {
        include: ['*'],
        exclude: []
    },
    zeroIndex: false
};
```


### Simple example

```javascript
const Hapi = require('hapi');

const server = new Hapi.Server();

// Add your connection

await server.register(require('hapi-pagination'))
```

### Example with options

```javascript
const Hapi = require('hapi');

const server = new Hapi.Server();

// Add your connection

const options = {
    query: {
      page: {
        name: 'the_page' // The page parameter will now be called the_page
      },
      limit: {
        name: 'per_page', // The limit will now be called per_page
        default: 10       // The default value will be 10
      }
    },
     meta: {
        location: 'body', // The metadata will be put in the response body
        name: 'metadata', // The meta object will be called metadata
        count: {
            active: true,
            name: 'count'
        },
        pageCount: {
            name: 'totalPages'
        },
        self: {
            active: false // Will not generate the self link
        },
        first: {
            active: false // Will not generate the first link
        },
        last: {
            active: false // Will not generate the last link
        }
     },
     routes: {
         include: ['/users', '/accounts', '/persons', '/'],
     }
};

await server.register({plugin: require('hapi-pagination'), options: options})
```
### Disable globally and activate pagination on specific routes

Global configuration:

```javascript
const Hapi = require('hapi');

const server = new Hapi.Server();

// Add your connection

const options = {
     routes: {
         include: [],  // Emptying include list will disable pagination
     }
};

await server.register({plugin: require('hapi-pagination'), options: options})
```
Activate on route level:

```javascript
config: {
    plugins: {
        pagination: {
            enabled: true
        }
    }
}
```
If you want to provide more examples, I'll accept a PR.

### Usage with Joi (route data validation)

You have two choices when you uses this plugin with Joi:

* You can simply add the `limit`, `page` and `pagination` to the query schema (with the names that you chose !).
* You can set the `allowUnknown` option to true.
  [See here](https://github.com/hapijs/joi/blob/master/API.md#validatevalue-schema-options-callback)

You don't need this if you don't need to validate anything !

#### Example

```javascript
validate: {
  query: {
    // Your other parameters ...
    limit: Joi.number().integer(),
    page: Joi.number().integer(),
    pagination: Joi.boolean()
  }
}

// OR

validate: {
  options: {
    allowUnknown: true
  }
  query: {
    // Your other parameters...
  }
}
```

## Tests

Make sure you have `lab` and `code` installed and run :

```
npm test
```
