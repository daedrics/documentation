---

title: 'HTTP Connector'
sidebar_position: 10

menu:
  main:
    identifier: "connect-ref-http-connector"
    parent: "connect-ref"

---

In Operaton Connect a `Connectors` class exists which automatically detects
every connector in the classpath. It can be used to get the HTTP connector
instance by its connector ID, which is `http-connector`.

```java
HttpConnector http = Connectors.getConnector(HttpConnector.ID);
```


# Configure Apache HTTP Client

Operaton Connect HTTP client uses the Apache HTTP client to make HTTP requests. Accordingly, it supports the same configuration options.

## Default Configuration

By default, the HTTP client uses Apache's default configuration and respects the [system properties that are supported by HTTP client](https://hc.apache.org/httpcomponents-client-4.5.x/current/httpclient/apidocs/org/apache/http/impl/client/HttpClientBuilder.html).
## Custom Configuration

If you want to reconfigure the client going beyond the default configuration options, e.g. you want to configure another connection manager, the easiest way is to register
a new connector configurator.

```java
package org.operaton.connect.example;

import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.operaton.connect.httpclient.impl.AbstractHttpConnector;
import org.operaton.connect.spi.ConnectorConfigurator;

public class HttpConnectorConfigurator implements ConnectorConfigurator<HttpConnector> {

  public Class<HttpConnector> getConnectorClass() {
    return HttpConnector.class;
  }

  public void configure(HttpConnector connector) {
    CloseableHttpClient client = HttpClients.custom()
      .setMaxConnPerRoute(10)
      .setMaxConnTotal(200)
      .build();
    ((AbstractHttpConnector) connector).setHttpClient(client);
  }

}
```

To enable auto detection of your new configurator please add a file called
`org.operaton.connect.spi.ConnectorConfigurator` to your
`resources/META-INF/services` directory with class name as content. For more
information see the [extending Connect](../reference/connect/extending-connect.md) section.

```
org.operaton.connect.example.HttpConnectorConfigurator
```

# Requests

## Create a Simple HTTP Request

The HTTP connector can be used to create a new request, set a HTTP method, URL,
content type and payload.

A simple GET request:

```java
http.createRequest()
  .get()
  .url("http://operaton.org")
  .execute();
```

A POST request with a content type and payload set:

```java
http.createRequest()
  .post()
  .url("http://operaton.org")
  .contentType("text/plain")
  .payload("Hello World!")
  .execute();
```

The HTTP methods PUT, DELETE, PATCH, HEAD, OPTIONS, TRACE
are also available.


## Adding HTTP Headers to a Request

To add own headers to the HTTP request the method `header` is
available.

```java
HttpResponse response = http.createRequest()
  .get()
  .header("Accept", "application/json")
  .url("http://operaton.org")
  .execute();
```

## Enabling HTTP Response Error Handling

By default, the HTTP connector does not seamlessly handle 4XX and 5XX related response errors during HTTP call.
To activate the handling of these errors without additional scripting, set the `throw-http-error` property to `TRUE` via the `configOption` method. Once enabled, the client will throw an exception in case of http response errors (status code 400-599).

```java
HttpResponse response = http.createRequest()
  .get()
  .configOption("throw-http-error", "TRUE")
  .url("http://operaton.org")
  .execute();
```

## Using the Generic API

Besides the configuration methods also a generic API exists to
set parameters of a request. The following parameters are
available:

<table class="table table-striped">
  <tr>
    <th>Parameter</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>method</td>
    <td>Sets the HTTP method of the request</td>
  </tr>
  <tr>
    <td>url</td>
    <td>Sets the URL of the request</td>
  </tr>
  <tr>
    <td>headers</td>
    <td>Contains a map of the configured HTTP headers of the request</td>
  </tr>
  <tr>
    <td>payload</td>
    <td>Sets the payload of the request</td>
  </tr>
</table>

This can be used as follows:

```java
HttpRequest request = http.createRequest();
request.setRequestParameter("method", "GET");
request.setRequestParameter("url", "http://operaton.org");
request.setRequestParameter("payload", "hello world!");
```

# Response

A response contains the status code, response headers and body.

```java
Integer statusCode = response.getStatusCode();
String contentTypeHeader = response.getHeader("Content-Type");
String body = response.getResponse();
```

After the response was processed it should be closed.

```java
response.close()
```

## Using the Generic API

Besides the response methods a generic API is provided
to gather the response parameters. The following parameters
are available:

<table class="table table-striped">
  <tr>
    <th>Parameter</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>statusCode</td>
    <td>Contains the status code of the response</td>
  </tr>
  <tr>
    <td>headers</td>
    <td>Contains a map with the HTTP headers of the response</td>
  </tr>
  <tr>
    <td>response</td>
    <td>Contains the response body</td>
  </tr>
</table>

This can be used as follows:

```java
response.getResponseParameter("statusCode");
response.getResponseParameter("headers");
response.getResponseParameter("response");
```
