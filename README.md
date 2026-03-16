# Http Request

A simple convenience library for using a [HttpURLConnection](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html)
to make requests and access the response.

This library is available under the [MIT License](https://opensource.org/licenses/MIT).

## About

**http-request** is a lightweight, zero-dependency library providing a fluent API for HTTP requests using Java's built-in `HttpURLConnection`. This is a modernized fork of Kevin Sawicki's original project, maintained by [josepacelli](https://github.com/josepacelli) and published to Maven Central with Java 1.8+ support.

## Installation

The http-request library is available from [Maven Central](https://central.sonatype.com/artifact/io.github.josepacelli/http-request).

```xml
<dependency>
  <groupId>io.github.josepacelli</groupId>
  <artifactId>http-request</artifactId>
  <version>1.0.0</version>
</dependency>
```

**Not using Maven?** Simply copy the [HttpRequest](https://raw.githubusercontent.com/josepacelli/http-request/master/lib/src/main/java/io/github/josepacelli/http/HttpRequest.java)
class into your project, update the package declaration to `io.github.josepacelli.http`, and you're all set.

**Javadocs** are available in the project's javadoc JAR or generated with `mvn javadoc:javadoc`.

## FAQ

### Why was this written?

This library was written to make HTTP requests simple and easy when using a `HttpURLConnection`.

Libraries like [Apache HttpComponents](https://hc.apache.org) are great, but sometimes you prefer simplicity or need to minimize dependencies (especially for environments like Android). This library provides a fluent API for building HTTP requests with support for common patterns like multipart requests, authentication, compression, and more—all without external dependencies.

**Bottom line:** Improve the usability of `HttpURLConnection` with minimal overhead.

### What are the dependencies?

**Zero production dependencies.** The library is intentionally designed as a single main class with inner static classes. This makes it:
- Lightweight and easy to include in any project
- Simple to copy-paste the single class if needed
- Ideal for Android and other constrained environments

The test suite uses [Jetty 9.4](https://www.eclipse.org/jetty/) to test against a real HTTP server implementation, but this is not required for production use.

### Java version requirements

This library targets **Java 1.8+**. It uses only JDK built-in classes and is compatible with Java 8 and later.

### How are exceptions managed?

The `HttpRequest` class does not throw any checked exceptions, instead all low-level
exceptions are wrapped up in a `HttpRequestException` which extends `RuntimeException`.
You can access the underlying exception by catching `HttpRequestException` and calling
`getCause()` which will always return the original `IOException`.

### Are requests asynchronous?

**No**.  The underlying `HttpUrlConnection` object that each `HttpRequest`
object wraps has a synchronous API and therefore all methods on `HttpRequest`
are also synchronous.

Therefore it is important to not use an `HttpRequest` object on the main thread
of your application.

Here is a simple Android example of using it from an
[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html):

```java
private class DownloadTask extends AsyncTask<String, Long, File> {
  protected File doInBackground(String... urls) {
    try {
      HttpRequest request =  HttpRequest.get(urls[0]);
      File file = null;
      if (request.ok()) {
        file = File.createTempFile("download", ".tmp");
        request.receive(file);
        publishProgress(file.length());
      }
      return file;
    } catch (HttpRequestException exception) {
      return null;
    }
  }

  protected void onProgressUpdate(Long... progress) {
    Log.d("MyApp", "Downloaded bytes: " + progress[0]);
  }

  protected void onPostExecute(File file) {
    if (file != null)
      Log.d("MyApp", "Downloaded file to: " + file.getAbsolutePath());
    else
      Log.d("MyApp", "Download failed");
  }
}

new DownloadTask().execute("http://google.com");
```

## Examples

### Perform a GET request and get the status of the response

```java
int response = HttpRequest.get("http://google.com").code();
```

### Perform a GET request and get the body of the response

```java
String response = HttpRequest.get("http://google.com").body();
System.out.println("Response was: " + response);
```

### Print the response of a GET request to standard out

```java
HttpRequest.get("http://google.com").receive(System.out);
```

### Adding query parameters

```java
HttpRequest request = HttpRequest.get("http://google.com", true, 'q', "baseball gloves", "size", 100);
System.out.println(request.toString()); // GET http://google.com?q=baseball%20gloves&size=100
```

### Using arrays as query parameters

```java
int[] ids = new int[] { 22, 23 };
HttpRequest request = HttpRequest.get("http://google.com", true, "id", ids);
System.out.println(request.toString()); // GET http://google.com?id[]=22&id[]=23
```

### Working with request/response headers

```java
String contentType = HttpRequest.get("http://google.com")
                                .accept("application/json") //Sets request header
                                .contentType(); //Gets response header
System.out.println("Response content type was " + contentType);
```

### Perform a POST request with some data and get the status of the response

```java
int response = HttpRequest.post("http://google.com").send("name=kevin").code();
```

### Authenticate using Basic authentication

```java
int response = HttpRequest.get("http://google.com").basic("username", "p4ssw0rd").code();
```

### Perform a multipart POST request

```java
HttpRequest request = HttpRequest.post("http://google.com");
request.part("status[body]", "Making a multipart request");
request.part("status[image]", new File("/home/kevin/Pictures/ide.png"));
if (request.ok())
  System.out.println("Status was updated");
```

### Perform a POST request with form data

```java
Map<String, String> data = new HashMap<String, String>();
data.put("user", "A User");
data.put("state", "CA");
if (HttpRequest.post("http://google.com").form(data).created())
  System.out.println("User was created");
```

### Copy body of response to a file

```java
File output = new File("/output/request.out");
HttpRequest.get("http://google.com").receive(output);
```
### Post contents of a file

```java
File input = new File("/input/data.txt");
int response = HttpRequest.post("http://google.com").send(input).code();
```

### Using entity tags for caching

```java
File latest = new File("/data/cache.json");
HttpRequest request = HttpRequest.get("http://google.com");
//Copy response to file
request.receive(latest);
//Store eTag of response
String eTag = request.eTag();
//Later on check if changes exist
boolean unchanged = HttpRequest.get("http://google.com")
                               .ifNoneMatch(eTag)
                               .notModified();
```

### Using gzip compression

```java
HttpRequest request = HttpRequest.get("http://google.com");
//Tell server to gzip response and automatically uncompress
request.acceptGzipEncoding().uncompress(true);
String uncompressed = request.body();
System.out.println("Uncompressed response is: " + uncompressed);
```

### Ignoring security when using HTTPS

```java
HttpRequest request = HttpRequest.get("https://google.com");
//Accept all certificates
request.trustAllCerts();
//Accept all hostnames
request.trustAllHosts();
```

### Configuring an HTTP proxy

```java
HttpRequest request = HttpRequest.get("https://google.com");
//Configure proxy
request.useProxy("localhost", 8080);
//Optional proxy basic authentication
request.proxyBasic("username", "p4ssw0rd");
```

### Following redirects

```java
int code = HttpRequest.get("http://google.com").followRedirects(true).code();
```

### Custom connection factory

Looking to use this library with [OkHttp](https://github.com/square/okhttp)?
Read [here](https://gist.github.com/JakeWharton/5797571).

```java
HttpRequest.setConnectionFactory(new ConnectionFactory() {

  public HttpURLConnection create(URL url) throws IOException {
    if (!"https".equals(url.getProtocol()))
      throw new IOException("Only secure requests are allowed");
    return (HttpURLConnection) url.openConnection();
  }

  public HttpURLConnection create(URL url, Proxy proxy) throws IOException {
    if (!"https".equals(url.getProtocol()))
      throw new IOException("Only secure requests are allowed");
    return (HttpURLConnection) url.openConnection(proxy);
  }
});
```

## License

MIT License - see [LICENSE.md](LICENSE.md) for details.

## About this Fork

This is a modernized fork of Kevin Sawicki's [http-request](https://github.com/kevinsawicki/http-request) library, maintained and published to Maven Central by [Jose Pacelli](https://github.com/josepacelli).

**Updates in this fork:**
- Java 1.8+ target (originally Java 1.5)
- Published to Maven Central under `io.github.josepacelli:http-request`
- Jetty dependency updated to 9.4+ for testing
- Package namespace: `io.github.josepacelli.http`
- Maintained and supported on GitHub

## Contributors

**Original Project:**
* [Kevin Sawicki](https://github.com/kevinsawicki) - Original author and creator

**Active Maintainers:**
* [Jose Pacelli](https://github.com/josepacelli) - Current maintainer, Maven Central publishing

**Original Contributors:**
* [Eddie Ringle](https://github.com/eddieringle)
* [Sean Jensen-Grey](https://github.com/seanjensengrey)
* [Levi Notik](https://github.com/levinotik)
* [Michael Wang](https://github.com/michael-wang)
* [Julien HENRY](https://github.com/henryju)
* [Benoit Lubek](https://github.com/BoD)
* [Jake Wharton](https://github.com/JakeWharton)
* [Oskar Hagberg](https://github.com/oskarhagberg)
* [David Pate](https://github.com/DavidTPate)
* [Anton Rieder](https://github.com/aried3r)
* [Jean-Baptiste Lièvremont](https://github.com/jblievremont)
* [Roman Petrenko](https://github.com/romanzes)
