Continuing on the work Ryan did, I have modified the KissMetricsClient to work with Google App Engine (GAE).  
GAE does not allow for the creation of Thread in it's environment hence the AsyncttpClient did not work.
However, I modified the KissMetricClient to use the Google App Engine fetchUrl class and substituted the
AsyncHttpClient.

The rest of the code is left untouched other that the test that exercised the KissMetricClient.

NOTE ON UNIT TESTS: I found out that GAE required a web app to make the URL call.  I had updated the unit
test however some fail because of this limitation in GAE.  At this time, I can't get around this.

Tony
  
=============================================

# work in progress

this is the start of a basic java client for kiss metrics...
its backed by AsyncHttpClient from https://github.com/sonatype/async-http-client


All you have to do is instantiate a KissMetricsClient - set the user's identity, set your api key then start making calls.

## simple use
    // since a AsyncHttpClient isn't passed in to KissMetricsClient it'll just make one
    KissMetricsClient client = new KissMetricsClient(apiKey, userIdentity);
    client.record("loggedin");
    client.record("purchased", new KissMetricsProperties().put("item", "latte"));
    client.alias("some alias for the dude");

## provide your own AsyncHttpClient

if you want to use connection pooling, a proxy etc... then you can pass in your own AsyncHttpClient.
more info can be found at https://github.com/sonatype/async-http-client (links at the top of their README).
but the basic idea would be something like this:

    Builder builder = new AsyncHttpClientConfig.Builder();
    builder.setCompressionEnabled(true)
        .setAllowPoolingConnection(true)
        .setRequestTimesout(30000)
        .build();
    AsyncHttpClient httpClient = new AsyncHttpClient(builder.build());
    KissMetricsClient client = new KissMetricsClient(apiKey, userIdentity, httpClient, false);
    client.alias("newAliashere");
    client.record("loggedin");

## its async...

as a result the record(), set() & alias() methods do NOT return the result of the API call to KissMetrics.
those method calls are fluent, so they actually return `this` which allows u to do stuff like:

    client.alias("newAliashere")
        .record("loggedin")
        .record("eat poop");

if you want the result of the HTTP call to KISS metrics you have to call `getResponse()` on the client.

    KissMetricsResponse resp = client.record("loggedin").getResponse();

our KissMetricsResponse class will get you the http status code and all the headers.
we don't bother storing the body b/c every time i've checked out the body it's a GIF.


## KissMetricsProperties

the properties you send to kiss metrics are abstracted by the `KissMetricsProperties` class.
the main reason we use this instead of a HashMap is
    * it has a fluent interface so you can do convenient stuff like
        props.put("key1", "value1").put("key2", "value2")
    * it restricts you to int, long, String, double, float and boolean (aka u cant pass absurd & giant objects to it)


# unit tests

they're actually functional tests... nothing's mocked.
you can use the Makefile to run the tests via:

    make KISS_API=<YOUR_API_KEY> test

so launch the test and look at your kiss metrics live console. if u wanna mock everything up have at it and send us a pull request