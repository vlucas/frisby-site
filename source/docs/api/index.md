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

#### expectHeaderToMatch( key, patterm )
Similar to expectHeaderContains, but use regular expression matching.
Uses Jasmine's toMatch internally.

```javascript
frisby.create('Ensure response has image/something in the Content-Type header')
  .get('http://httpbin.org/get')
    .expectHeaderContains('Content-Type', '^image/.+')
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
To use expectJSON and expectJSONContains to test all objects in an array, simply
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
    .expectJSON('?', {
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

#### expectMaxResponseTime( milliseconds )
Ensure the response is received before a specified delay.

```javascript
frisby.create('Ensure response is fast enough')
  .get('http://httpbin.org/get')
    .expectMaxResponseTime(5)
.toss()
```

### Helpers

#### after()
Callback function to run after test is completed. Can be used to run one test
after another.

```javascript
frisby.create('First test')
  .get('http://httpbin.org/get?foo=bar')
  .after(function(err, res, body) {

    frisby.create('Second test, run after first is completed')
      .get('http://httpbin.org/get?bar=baz')
    .toss()

  })
.toss()
```

#### afterJSON()
Callback function to run after test is completed. Helper to also automatically
convert response body to JSON.

```javascript
frisby.create('First test')
  .get('http://httpbin.org/get?foo=bar')
  .afterJSON(function(json) {

    // Now you can use 'json' in additional requests
    frisby.create('Second test, run after first is completed')
      .get('http://httpbin.org/get?bar=' + json.args.foo)
    .toss()

  })
.toss()
```

#### retry( count, delay )
Define how many times you want to retry a failing test.
The delay period between each retry has to be specified in milliseconds.

```javascript
frisby.create('This fails on purpose, go ahead and retry...')
  .get('http://httpbina.org/get?foo=bar&bar=baz')
    .expectStatus(400)
    .retry(5, 1000)
.toss()
```

#### waits( delay )
Define a delay period ( in milliseconds ) before executing the test.

```javascript
frisby.create('Wait before I start...')
  .get('http://httpbina.org/get?foo=bar&bar=baz')
    .expectStatus(200)
    .waits(500)
.toss()
```

#### exceptionHandler( function )
Define an exception handler function, this function catches any exception thrown
by 'expects' helpers. It receives one argument which is the error thrown.

```javascript
frisby.create('Catch me')
  .get('http://httpbin.org/get')
    .expectHeader('I do no exist', 'fail')
    .exceptionHandler(function(e) {
      self.abort = true;
    })
.toss()
```

### Headers

#### addHeader(header, content)
Add HTTP header by key and value.

#### addHeaders(headers)
Add group of HTTP headers together. Accept object as headers parameter, each key is used as header key.

#### removeHeader(key)
Remove HTTP header from outgoing request by key.

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

#### inspectRequest()
Dumps raw HTTP request sent by Frisby.

```javascript
frisby.create('Inspect the request object just after it is executed')
  .post('http://httpbin.org/get?foo=bar&bar=baz', { some: 'test data' })
    .inspectRequest()
.toss()
```

```
// Console output
{ json: false,
  uri: 'http://httpbin.org/get?foo=bar&bar=baz',
  body: 'some=test%20data',
  method: 'POST',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  inspectOnFailure: false,
  baseUri: '',
  timeout: 5000 }
```

#### inspectHeaders()
Dumps raw HTTP response headers.

```javascript
frisby.create('Just a quick inspection of the HTTP headers response')
  .get('http://httpbin.org/get')
    .inspectHeaders()
.toss()
```

```
// Console output
{ server: 'nginx',
  date: 'Mon, 01 Jan 1970 00:00:00 GMT',
  'content-type': 'application/json',
  'content-length': '244',
  connection: 'keep-alive',
  'access-control-allow-origin': '*',
  'access-control-allow-credentials': 'true' }
```

#### inspectStatus()
Dumps HTTP status code received from server.

```javascript
frisby.create('Just a quick inspection of the HTTP status code response')
  .get('http://httpbin.org/get')
    .inspectStatus()
.toss()
```

```
// Console output (even if little)
200
```

### Send Raw JSON or POST Body
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

### Get GZIP Response Body
When the API under test returns a GZIP response, use `{ gzip : true }`
```
frisby.create('Gzip Sample')
.get('http://httpbin.org/gzip', {gzip : true})
.inspectBody()
.expectHeader('content-encoding', 'gzip')
.toss();
```
Console output `with` { gzip : true }
```
{                                                       
  "gzipped": true,                                      
  "headers": {                                          
    "Cache-Control": "max-stale=0",                     
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org"                               
  },                                                    
  "method": "GET",                                      
  "origin": "100.100.100.1"                             
}                                                       
```

Console output `without` { gzip : true }
```
▼ �♠rW☻�5�M
�0►��=E���♠⌂►�U◄=@/►۱   ���Ni���&�������¶���mC���↓�◄6,3♥��8$��3�J7♠D��"��y�g1�vp)��ɥ¶�'Q�☻��♫��F�E���4M≱↨ct�l�Ͽyǁ�a���z���)Y⌂kz �y �]���1����Ծ��x�jw��↨k�☺Pd�<�
```
