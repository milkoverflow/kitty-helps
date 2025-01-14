# Golang

- [Serving a site](#serving-a-site)

## Serving a site

```go
package main

import (
	"fmt"
	"net/http"
)

func root(w http.ResponseWriter, req *http.Request) {
	http.ServeFile(w, req, "./views/index.html")
}

func say_hi(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(w, "Hello There!")
}

func main() {
	// Serve one file with middleware
	http.HandleFunc("/", root)
	// Create an API endpoint
	http.HandleFunc("/api/say_hi", say_hi)
	// Serve static files
	http.Handle("/static/", http.StripPrefix("/static", http.FileServer(http.Dir("./static"))))

	http.ListenAndServe(":8090", nil)
}

```
