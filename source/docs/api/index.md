title: Frisby.js API Documentation
date: 2014-04-14 10:47:54
---
The main Frisby API methods

### Expectations

Expectation helper methods are used after the create method, and before the
toss method that generates the final Jasmine test spec. These helpers make
testing API response bodies and headers easy with minimal time and effort.

#### expectStatus( code )
Tests that HTTP Status code equals expectation

```javascript
frisby.create('Ensure we are dealing with a teapot')
  .get('http://httpbin.org/status/418')
    .expectStatus(418)
.toss()
```

#### expectHeader( key, content )
Tests that HTTP response contains a specific header with exact content. Both
key and content comparisons are case-insensitive (converted to lowercase
    internally). Uses Jasmine's toEqual matcher (strict).

```javascript
frisby.create('Ensure response has a proper JSON Content-Type header')
  .get('http://httpbin.org/get')
    .expectHeader('Content-Type', 'application/json')
.toss();
```

#### expectHeaderContains( key, content )
Less strict version of expectHeader. Checks to ensure content exists within
header. Uses Jasmine's toContain matcher internally.

```javascript
frisby.create('Ensure response has JSON somewhere in the Content-Type header')
  .get('http://httpbin.org/get')
    .expectHeaderContains('Content-Type', 'json')
.toss();
```

#### expectJSON( [path], json )
Tests that response body is JSON and contains the provided JSON keys and values
in the response.

```javascript
frisby.create('Ensure test has foo and bar')
  .get('http://httpbin.org/get?foo=bar&bar=baz')
    .expectJSON({
      args: {
        foo: 'bar',
        bar: 'baz'
      }
    })
.toss()
```

#### expectJSONTypes( [path], json )
Identical syntax as expectJSON, but tests the types of the JSON values instead
of the content.

```javascript
frisby.create('Ensure response has proper JSON types in specified keys')
  .post('http://httpbin.org/post', {
      arr: [1, 2, 3, 4],
      foo: "bar",
      bar: "baz",
      answer: 42
    })
    .expectJSONTypes('args', {
      arr: Array,
      foo: String,
      bar: String,
      answer: Number
    })
.toss()
```

#### Using Paths with expectJSON and expectJSONTypes
In addition to nested JSON, both expectJSON and expectJSONTypes accept a path
as the first parameter as a useful shortcut. The path parameter can be a nested
path separated by periods, like 'args.foo.mypath', or a simple path like
'results'.

```javascript
frisby.create('Ensure test has foo and bar')
  .get('http://httpbin.org/get?foo=bar&bar=baz')
    .expectJSON('args', {
      foo: 'bar',
      bar: 'baz'
    })
.toss()
```

##### Testing Arrays of Objects
In addition to accepting a nested path, both expectJSON and expectJSONContains
accept two special variations for testing arrays of objects. One is for testing
all of the objects in an array, and the other is for testing for at least one
object in an array.

##### All Objects in an Array
To use expectJSON and expectJSONContains test all objects in an array, simply
append a path with an asterisk character, so the path looks like
args.path.myarray.* if the array is at the root level, just use '*' as the
path.

This path mode is often combined with expectJSONTypes to ensure each item in an
array contains all the proper keys and types required.

```javascript
frisby.create('Ensure each tweet has base attributes')
  .get('https://api.twitter.com/1/statuses/user_timeline.json?screen_name=brightbit')
  .expectStatus(200)
  .expectHeaderContains('content-type', 'application/json')
  .expectJSONTypes('*', {
    id_str: String,
    retweeted: Boolean,
    in_reply_to_screen_name: function(val) { expect(val).toBeTypeOrNull(String); }, // Custom matcher callback
    user: {
      verified: Boolean,
      location: String,
      url: String
    }
  })
.toss();
```

Note that a custom callback is used in this example. When a callback is used,
 Frisby does not run any match for that key. The callback either has to
 return true/false (pass/fail), or can use custom Jasmine matchers
 directly. Custom callbacks are an easy way to use your own Jasmine
 matchers or comparison methods in conjunction with Frisby.

##### One Object in an Array
To use expectJSON and expectJSONContains to test for one object in an array
that matches, simply append a path with an question mark, so the path looks
like args.path.myarray.? if the array is at the root level, just use '?' as the
path.

This path mode is most useful when combined with expectJSON, because it is
often used when you know a specific item is supposed to exist in a returned
collection.

```javascript
frisby.create('Ensure Twitter has at least one list that is "NBA"')
  .get('https://api.twitter.com/1/lists/all.json?screen_name=twitter')
    .expectStatus(200)
    .expectHeader('content-type', 'application/json')
    .expectJSONTypes('?', {
      name: "NBA",
      full_name: "@twitter/nba-7",
      id_str: "42840851",
      description: "All verified NBA players on Twitter",
      mode: "public"
    })
.toss();
```

#### expectBodyContains( content )

Test the HTTP response body for a given content string. This method does NOT
parse the response body as JSON, making it very useful for testing HTML, text,
or other content types. Uses Jasmine's toContain matcher internally.

```javascript
frisby.create('Ensure this is *actually* a real teapot, not some imposter coffee pot')
  .get('http://httpbin.org/status/418')
    .expectStatus(418)
    .expectBodyContains('teapot')
.toss()
```

#### expectJSONLength( [path], length )
Tests given path or full JSON response for specified length. When used on
objects, the number of keys are counted. When used on other JavaScript types
like Arrays or Strings, the native length property is used for comparison.

```javascript
frisby.create('Ensure "bar" really is only 3 characters... because you never know...')
  .get('http://httpbin.org/get?foo=bar&bar=baz')
    .expectJSONLength('args.foo', 3)
.toss()
```

### Inspectors

Inspector helpers are useful for viewing details about the HTTP response when
the test does not pass, or has trouble for some reason. They are also useful
for debugging the API itself as a more user-friendly alternative to curl.

#### inspectJSON()
Dumps parsed JSON body in console using the node.js pretty printing utility
method util.inspect.

```javascript
// Test
frisby.create('Just a quick inspection of the JSON HTTP response')
  .get('http://httpbin.org/get?foo=bar&bar=baz')
    .inspectJSON()
.toss()
```

```
// Console output
{ url: 'http://httpbin.org/get?foo=bar&bar=baz',
  headers:
   { 'Content-Length': '',
     'X-Forwarded-Port': '80',
     Connection: 'keep-alive',
     Host: 'httpbin.org',
     Cookie: '',
     'Content-Type': 'application/json' },
  args: { foo: 'bar', bar: 'baz' },
  origin: '127.0.0.1' }
```

#### inspectBody()
Dumps the raw response body without any parsing or markup added.

```javascript
// Test
frisby.create('Very useful for HTML, text, or raw output')
  .get('http://asciime.heroku.com/generate_ascii?s=Frisby.js')
    .inspectBody()
.toss()
```

```
// Console Output
  ______    _     _             _
 |  ____|  (_)   | |           (_)
 | |__ _ __ _ ___| |__  _   _   _ ___
 |  __| '__| / __| '_ \| | | | | / __|
 | |  | |  | \__ \ |_) | |_| |_| \__ \
 |_|  |_|  |_|___/_.__/ \__, (_) |___/
                         __/ |_/ |
                        |___/|__/
```

#### after()
Callback function to run after test is completed
```javascript
this.after(function(err, res, body) {
  console.log(self.currentRequestFinished.req);
});
```

#### afterJSON()
Callback function to run after test is completed. Helper to also automatically
convert response body to JSON.

#### Send Raw JSON or POST Body
By default, Frisby sends POST and PUT requests as
`application/x-www-form-urlencoded` parameters. If you want to send a raw
request body or actual JSON, use `{ json: true }` as the third argument (object
literal of options).

```javascript
frisby.create('Post JSON string as body')
    .post('http://httpbin.org/post', {
        arr: [1, 2, 3, 4],
        foo: "bar",
        bar: "baz",
        answer: 42
    }, {json: true})
    .expectHeaderContains('Content-Type', 'json')
.toss()
```

