---
title: "Metrics with the channels"
date: 2020-05-07T01:55:51+02:00
draft: true
---
Metrics allows us to collect the telemetry of the system
and use that information for further research and investigation.

Some system has millions metrics, other have no metrics at all.
Let's say that all modern application has metrics.
That means that beside the business logic
we need to add additional operations into the code to create, update, collect and dump the metrics.

By the nature, any metric should be thread-safe, otherwise we will get a race condition and end with incorect information.
To archive such a behavior, the most common approach is to use a mutex primitive. Let's see the basic example


```go
package metrics

import "sync"

var metricsMutex sync.Mutex
var requestsCounter int64

func RequestsIncrement() {
	metricsMutex.Lock()
	defer metricsMutex.Unlock()
	requestsCounter++
}
```


   
Here, we going to count all requests passed the system(say web server).
Once, the system get a new request
it starts the request handler in a goroutine and
goes back to be ready to accept new requests.

Let's create a simple client for our metrics package.

```go
func handleRequest() {
	metrics.RequestsIncrement()
	time.Sleep(1 * time.Millisecond) // processing time
}

func main() {
	for i := 0; i < 5; i++ {
		go handleRequest()
	}
	time.Sleep(1 * time.Second) // we need to wait while all goroutine finished
	metrics.Dump()
}
```

In a for loop, we start 5 requests' handlers and
wait while all handlers finish thier job.
At the end we dump the metric to stdout. When you run that code you will get the next output:

```bash
go run /home/mshogin/go/src/github.com/mshogin/mshogin.github.io/demo/blog_golang_metrics.go
requests.count=5
```

The result above is what we expect to get.

Let's adjust metrics package and remove mutex and see what happen then.

```go
func RequestsIncrement() {
	requestsCounter++
}
```

Then run the our programm several times

