golang library useful for logging API requests.

# Usage

See [api_logger/api_logger.go](api_logger/api_logger.go)

```go
package main

import (
	"log"
	"net/http"
	"os"
	"time"
)

func main() {
	client := http.Client{
		Transport: httplogger.NewLoggedTransport(http.DefaultTransport, newLogger()),
	}

	client.Get("http://infograins.com")
}

type httpLogger struct {
	log *log.Logger
}

func newLogger() *httpLogger {
	return &httpLogger{
		log: log.New(os.Stderr, "log - ", log.LstdFlags),
	}
}

func (l *httpLogger) LogRequest(req *http.Request) {
	l.log.Printf(
		"Request %s %s",
		req.Method,
		req.URL.String(),
	)
}

func (l *httpLogger) LogResponse(req *http.Request, res *http.Response, err error, duration time.Duration) {
	duration /= time.Millisecond
	if err != nil {
		l.log.Println(err)
	} else {
		l.log.Printf(
			"Response method=%s status=%d durationMs=%d %s",
			req.Method,
			res.StatusCode,
			duration,
			req.URL.String(),
		)
	}
}
```

Output:

```
% go run api_logger/api_logger.go
log - 2014/08/17 02:19:19 Request GET http://infograins.com
log - 2014/08/17 02:19:19 Response method=GET status=302
durationMs=85 http://infograins.com
log - 2014/08/17 02:19:19 Request GET
http://www.infograins.com/
log - 2014/08/17 02:19:20 Response method=GET status=200
durationMs=138
http://www.infograins.com/
```
