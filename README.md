# ktor-simple-demo

ktor-simple-demo is your new project powered by [Ktor](http://ktor.io) framework.

<img src="https://repository-images.githubusercontent.com/40136600/f3f5fd00-c59e-11e9-8284-cb297d193133" alt="Ktor" width="100" style="max-width:20%;">

Company website: example.com Ktor Version: 1.5.2 Kotlin Version: 1.4.10
BuildSystem: [Gradle with Kotlin DSL](https://docs.gradle.org/current/userguide/kotlin_dsl.html)

# Ktor Documentation

Ktor is a framework for quickly creating web applications in Kotlin with minimal effort.

* Ktor project's [Github](https://github.com/ktorio/ktor/blob/master/README.md)
* Getting started with [Gradle](http://ktor.io/quickstart/gradle.html)
* Getting started with [Maven](http://ktor.io/quickstart/maven.html)
* Getting started with [IDEA](http://ktor.io/quickstart/intellij-idea.html)

Selected Features:

* [Routing](#routing-documentation-jetbrainshttpswwwjetbrainscom)
* [HTML DSL](#html-dsl-documentation-jetbrainshttpswwwjetbrainscom)

## Routing Documentation ([JetBrains](https://www.jetbrains.com))

Allows to define structured routes and associated handlers.

### Description

Routing is a feature that is installed into an Application to simplify and structure page request handling. This page
explains the routing feature. Extracting information about a request, and generating valid responses inside a route, is
described on the requests and responses pages.

```application.install(Routing) {
    get("/") {
        call.respondText("Hello, World!")
    }
    get("/bye") {
        call.respondText("Good bye, World!")
    }

```

`get`, `post`, `put`, `delete`, `head` and `options` functions are convenience shortcuts to a flexible and powerful
routing system. In particular, get is an alias to `route(HttpMethod.Get, path) { handle(body) }`, where body is a lambda
passed to the get function.

### Usage

## Routing Tree

Routing is organized in a tree with a recursive matching system that is capable of handling quite complex rules for
request processing. The Tree is built with nodes and selectors. The Node contains handlers and interceptors, and the
selector is attached to an arc which connects another node. If selector matches current routing evaluation context, the
algorithm goes down to the node associated with that selector.

Routing is built using a DSL in a nested manner:

```kotlin
route("a") { // matches first segment with the value "a"
  route("b") { // matches second segment with the value "b"
     get {…} // matches GET verb, and installs a handler
     post {…} // matches POST verb, and installs a handler
  }
}
```

```kotlin
method(HttpMethod.Get) { // matches GET verb
   route("a") { // matches first segment with the value "a"
      route("b") { // matches second segment with the value "b"
         handle { … } // installs handler
      }
   }
}
```kotlin
route resolution algorithms go through nodes recursively discarding subtrees where selector didn't match.

Builder functions:
* `route(path)` – adds path segments matcher(s), see below about paths
* `method(verb)` – adds HTTP method matcher.
* `param(name, value)` – adds matcher for a specific value of the query parameter
* `param(name)` – adds matcher that checks for the existence of a query parameter and captures its value
* `optionalParam(name)` – adds matcher that captures the value of a query parameter if it exists
* `header(name, value)` – adds matcher that for a specific value of HTTP header, see below about quality

## Path
Building routing tree by hand would be very inconvenient. Thus there is `route` function that covers most of the use cases in a simple way, using path.

`route` function (and respective HTTP verb aliases) receives a `path` as a parameter which is processed to build routing tree. First, it is split into path segments by the `/` delimiter. Each segment generates a nested routing node.

These two variants are equivalent:

```kotlin
route("/foo/bar") { … } // (1)

route("/foo") {
   route("bar") { … } // (2)
}
```

### Parameters

Path can also contain parameters that match specific path segment and capture its value into `parameters` properties of
an application call:

```kotlin
get("/user/{login}") {
   val login = call.parameters["login"]
}
```

When user agent requests `/user/john` using `GET` method, this route is matched and `parameters` property will
have `"login"` key with value `"john"`.

### Optional, Wildcard, Tailcard

Parameters and path segments can be optional or capture entire remainder of URI.

* `{param?}` –- optional path segment, if it exists it's captured in the parameter
* `*` –- wildcard, any segment will match, but shouldn't be missing
* `{...}` –- tailcard, matches all the rest of the URI, should be last. Can be empty.
* `{param...}` –- captured tailcard, matches all the rest of the URI and puts multiple values for each path segment
  into `parameters` using `param` as key. Use `call.parameters.getAll("param")` to get all values.

Examples:

```kotlin
get("/user/{login}/{fullname?}") { … }
get("/resources/{path...}") { … }
```

## Quality

It is not unlikely that several routes can match to the same HTTP request.

One example is matching on the `Accept` HTTP header which can have multiple values with specified priority (quality).

```kotlin
accept(ContentType.Text.Plain) { … }
accept(ContentType.Text.Html) { … }
```

The routing matching algorithm not only checks if a particular HTTP request matches a specific path in a routing tree,
but it also calculates the quality of the match and selects the routing node with the best quality. Given the routes
above, which match on the Accept header, and given the request header `Accept: text/plain; q=0.5, text/html` will
match `text/html` because the quality factor in the HTTP header indicates a lower quality fortext/plain (default is 1.0)
.

The Header `Accept: text/plain, text/*` will match `text/plain`. Wildcard matches are considered less specific than
direct matches. Therefore the routing matching algorithm will consider them to have a lower quality.

Another example is making short URLs to named entities, e.g. users, and still being able to prefer specific pages
like `"settings"`. An example would be

* `https://twitter.com/kotlin` -– displays user `"kotlin"`
* `https://twitter.com/settings` -- displays settings page

This can be implemented like this:

```kotlin
get("/{user}") { … }
get("/settings") { … }
```

The parameter is considered to have a lower quality than a constant string, so that even if `/settings` matches both,
the second route will be selected.

### Options

No options()

## HTML DSL Documentation ([JetBrains](https://www.jetbrains.com))

Generate HTML using Kotlin code like a pure-core template engine

### Description

HTML DSL integrates the `kotlinx.html` library into Ktor and allows you to respond to a client with HTML blocks. With
HTML DSL, you can write pure HTML in Kotlin, interpolate variables into views, and even build complex HTML layouts using
templates.

### Usage

## Send HTML in Response

To send an HTML response, call the `ApplicationCall.respondHtml` method inside the required route:

```kotlin
get("/") {
    val name = "Ktor"
    call.respondHtml {
        head {
            title {
                +name
            }
        }
        body {
            h1 {
                +"Hello from $name!"
            }
        }
    }
}
```

In this case, the following HTML will be sent to the client:

```html
<head>
    <title>Ktor</title>
</head>
<body>
    <h1>Hello from Ktor!</h1>
</body>
```

To learn more about generating HTML using kotlinx.html, see
the [kotlinx.html wiki](https://github.com/Kotlin/kotlinx.html/wiki).

## Templates

In addition to generating plain HTML, Ktor provides a template engine that can be used to build complex layouts. You can
create a hierarchy of templates for different parts of an HTML page, for example, a root template for the entire page,
child templates for a page header and footer, and so on. Ktor exposes the following API for working with templates:

1. To respond with an HTML built based on a specified template, call the `ApplicationCall.respondHtmlTemplate` method.
2. To create a template, you need to implement the `Template` interface and override the `Template.apply` method
   providing HTML.
3. Inside a created template class, you can define placeholders for different content types:

* `Placeholder` is used to insert the content. `PlaceholderList` can be used to insert the content that appears multiple
  times (for example, list items).
* `TemplatePlaceholder` can be used to insert child templates and create nested layouts.

### Example

Let's see the example of how to create a hierarchical layout using templates. Imagine we have the following HTML:

```html
<body>
<h1>Ktor</h1>
<article>
    <h2>Hello from Ktor!</h2>
    <p>Kotlin Framework for creating connected systems.</p>
</article>
</body>
```

We can split the layout of this page into two parts:

* A root layout template for a page header and a child template for an article.
* A child template for the article content.

Let's implement these layouts step-by-step:

1. Call the `respondHtmlTemplate` method and pass a template class as a parameter. In our case, this is
   the `LayoutTemplate` class that should implement the `Template` interface:

```kotlin
get("/") {
    call.respondHtmlTemplate(LayoutTemplate()) {
        // ...
    }
}
```

Inside the block, we will be able to access a template and specify its property values. These values will substitute
placeholders specified in a template class. We'll create LayoutTemplate and define its properties in the next step.

2. A root layout template will look in the following way:

```kotlin
class LayoutTemplate: Template<HTML> {
    val header = Placeholder<FlowContent>()
    val content = TemplatePlaceholder<ContentTemplate>()
    override fun HTML.apply() {
        body {
            h1 {
                insert(header)
            }
            insert(ContentTemplate(), content)
        }
    }
}
```

The class exposes two properties:

* The `header` property specifies a content inserted within the h1 tag.
* The `content` property specifies a child template for article content.

3. A child template will look as follows:

```kotlin
class ContentTemplate: Template<FlowContent> {
    val articleTitle = Placeholder<FlowContent>()
    val articleText = Placeholder<FlowContent>()
    override fun FlowContent.apply() {
        article {
            h2 {
                insert(articleTitle)
            }
            p {
                insert(articleText)
            }
        }
    }
}
```

This template exposes the `articleTitle` and `articleText` properties, whose values will be inserted inside
the `article`.

4. Now we are ready to send HTML built using the specified property values:

```kotlin
get("/") {
    call.respondHtmlTemplate(LayoutTemplate()) {
        header {
            +"Ktor"
        }
        content {
            articleTitle {
                +"Hello from Ktor!"
            }
            articleText {
                +"Kotlin Framework for creating connected systems."
            }
        }
    }
}
```

### Options

No options()

# Reporting Issues / Support

Please use [our issue tracker](https://youtrack.jetbrains.com/issues/KTOR) for filing feature requests and bugs. If
you'd like to ask a question, we recommmend [StackOverflow](https://stackoverflow.com/questions/tagged/ktor) where
members of the team monitor frequently.

There is also community support on the [Kotlin Slack Ktor channel](https://app.slack.com/client/T09229ZC6/C0A974TJ9)

# Reporting Security Vulnerabilities

If you find a security vulnerability in Ktor, we kindly request that you reach out to the JetBrains security team via
our [responsible disclosure process](https://www.jetbrains.com/legal/terms/responsible-disclosure.html).

# Contributing

Please see [the contribution guide](CONTRIBUTING.md) and the [Code of conduct](CODE_OF_CONDUCT.md) before contributing.

TODO: contribution of features guide (link)