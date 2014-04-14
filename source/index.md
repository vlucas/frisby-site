title: REST API Testing
date: 2014-04-14 10:47:54
---

Frisby is a REST API testing framework built on node.js and Jasmine that makes
testing API endpoints easy, fast, and fun. Read below for a quick overview, or
check out the API documentation.

### Install Frisby
Frisby requires both node.js and NPM to be installed on your system, and is installable as an NPM package.

```
npm install -g frisby
```

### Write Tests
Frisby tests start with frisby.create with a description of the test followed by one of get, post, put, delete, or head, and ending with toss to generate the resulting jasmine spec test. Frisby has many built-in test helpers like expectStatus to easily test HTTP status codes, expectJSON to test expected JSON keys/values, and expectJSONTypes to test JSON value types, among many others.

```javascript
var frisby = require('frisby');

frisby.create('Get Brightbit Twitter feed')
  .get('https://api.twitter.com/1/statuses/user_timeline.json?screen_name=brightbit')
  .expectStatus(200)
  .expectHeaderContains('content-type', 'application/json')
  .expectJSON('0', {
    place: function(val) { expect(val).toMatchOrBeNull("Oklahoma City, OK"); }, // Custom matcher callback
    user: {
      verified: false,
      location: "Oklahoma City, OK",
      url: "http://brightb.it"
    }
  })
  .expectJSONTypes('0', {
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

Any of the Jasmine matchers can be used inside anonymous functions instead of
key values, and inside the after and afterJSON callbacks to perform additional
or custom tests on the response data.

[**Full API Documentation ►**](/docs/api)

### Run Tests
Frisby is built on top of the Jasmine BDD framework, and uses the jasmine-node
test runner to run spec tests.

#### Install jasmine-node
```
npm install -g jasmine-node
```

#### File naming conventions
Files must end with spec.js to run with jasmine-node.

Suggested file naming is to append the filename with \_spec, `like
mytests_spec.js` and `moretests_spec.js`

#### Run it from the CLI
```
cd your/project
jasmine-node spec/api/
```

#### Continuous Integration
The jasmine-node test runner has an option that generates test run reports in JUnit XML format. This format is compaitlble with most CI servers, including Hudson, Jenkins, Bamboo, Pulse, and others.

All you need to do is run the tests with the --junitreport flag in your CI server:

```
jasmine-node spec/api/ --junitreport
```
