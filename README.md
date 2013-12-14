# HttpRouter [![Build Status](https://travis-ci.org/julienschmidt/httprouter.png?branch=master)](https://travis-ci.org/julienschmidt/httprouter) [![Codebot](https://codebot.io/badge/github.com/julienschmidt/httprouter.png)](http://codebot.io/doc/pkg/github.com/julienschmidt/httprouter "Codebot") [![GoDoc](http://godoc.org/github.com/julienschmidt/httprouter?status.png)](http://godoc.org/github.com/julienschmidt/httprouter)

HttpRouter is a high performance HTTP request router
(also called *multiplexer* or just *mux* for short).

In contrast to the default mux of Go's net/http package, this router supports
variables in the routing pattern and matches against the request method.
It also scales better.

The router is optimized for best performance and a small memory footprint.
It scales well even with very long pathes and a large number of routes.
A compressing dynamic trie (radix tree) structure is used for efficient matching.

## Features
**Variables in your routing pattern:** Stop parsing the requested URL path, just give the path segment a name and the router delivers the value to you. Because of the design of the router, pattern variables are very cheap.

**Only explicit matches:** With other routers / *mux*es, like http.ServeMux, a requested URL path can be matched by multiple patterns. Therefore they have some awkward pattern priority rules, like *longest match* or *first registered, first matched*. By design, a request can only match exactly one or no route. 

**Stop caring about trailing slashes:** Choose the URL style you like, the router automatically redirects the client if a trailing slash is missing or if there is one extra. Of course it only does so, if the new path has a handler. If you don't like it, you can turn off this behavior.

**No more server crashes:** You can set a PanicHandler to deal with panics. The router then recovers and lets the PanicHandler log what happened and deliver a nice error page.

Of course you can also set a custom NotFound handler and serve files.

## Usage
Let's start with a trivial example:
```go
package main

import (
    "fmt"
    "github.com/julienschmidt/httprouter"
    "net/http"
    "log"
)

func Index(w http.ResponseWriter, r *http.Request, _ map[string]string) {
    fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, vars map[string]string) {
    fmt.Fprintf(w, "hello, %s!\n", vars["name"])
}

func main() {
    router := httprouter.New()
    router.GET("/", Index)
    router.GET("/hello/:name", Hello)

    log.Fatal(http.ListenAndServe(":12345", router))
}
```

### Named parameters
As you can see, `:name` is a *named parameter*.
The values are passed in map, therefore the value of `:name` is available in `vars["name"]`.

Named parameters only match a single path segment.
If we had the pattern `/user/:user`, only requests with a URL path like `/user/gordon` or
`/user/you` would be matched by this pattern, not `/user/gordon/profile` or `/user/`.

**Note:** Since this router has only explicit matches, you can not register static routes and paramters for the same path segment. For example you can not register the patterns `/user/new` and `/user/:user` at the same time.

### CatchAlls
The second type are *catchAll*s and have the form `*name`. Like the name suggest, they match everything.
Therefore they must always be at the **end** of the pattern.

The pattern `/src/*filepath` would match `/src/`, `/src/somefile.go`, `/src/subdir/somefile.go` and so on. 
