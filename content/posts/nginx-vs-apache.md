---
title: "Web Server Architecture: NGINX vs Apache"
date: 2018-12-11T17:44:48-05:00
draft: false
---


Apache and NGINX are the two most popular web servers in existence.  However, their architectures are radically different, and this has consequences for how well they perform under load.  In this post, we'll go over the architecture for each, their performance characteristics, and which you should choose for your next project!

To start, we should first understand what a web server actually does most of the time.  If we think about it, the bare essence of a web server is to receive a request from a client, processes the request, and then return a response based on the result of processing.  For a web server, this generally happens within the context of an HTTP request.  Not surprisingly, this request processing pipeline can be implemented in many different ways. 

For the sake of example, let's assume we have two servers, one running Apache and one running NGINX, and they both are serving a Python web app.

### Apache ###

Apache implements what is called a blocking multi-threaded architecture.  In this model, for each client connection to the server, a new thread is created by Apache.  The thread is blocking, which is to say the thread waits around until it gets a request from the client.  Once the request is received, the thread will invoke the Python web app and pass it the request.  The thread will then wait until the Python web app returns a response.  

When the web app returns a response, the thread will return the response to the client.  The thread will then sit and wait for a new request from the client.

Here's our high-level algorithm:

```
[Apache]
    for each new client connection:
        invoke new [Thread]
```
```
[Thread]
    1. Wait for request from client
    2. Pass request to Python web app
    3. Wait for Python web app to return a response
    4. Return response to client
    5. Goto 1
```

Here's an illustration of the process:

<img src="/images/posts/nginx-vs-apache/apache-arch.png" alt="Apache Architecture" />

### NGINX ###

NGINX implements a non-blocking single-threaded architecture.  More specifically, it implements the reactor pattern.  In this pattern, NGINX has a single thread that handles *all* client connections, unlike the one-thread per connection model of Apache.  When NGINX starts, it creates a single thread that sits and wait for the OS to notify it when there's a client request (there's a OS-level call that makes this easy called `epoll_wait()`).

Once notified, NGINX then takes the request and passes it to the Python web app.  NGINX then sits idle once again.  However, while NGINX waits, the OS might inform NGINX of other requests waiting in the queue.  In this case, while NGINX is waiting on a response from the Python web app, it will handle the other requests in the same manner (hence it is non-blocking).  Once the OS notifies NGINX of a response from the Python web app from the initial request, NGINX passes that response back to the client.

Here's our high-level algorithm:

```
[Nginx]
    1. Wait until OS notifies us of a client request or a response from Python web app using epoll_wait()
    2. When new request or response available:
        If it is a request from client:
            dispatch request to Python web app
        If it is a response from Python web app:
            dispatch response to client
    3. Goto 1
```

Here's an illustration of the process:

<img src="/images/posts/nginx-vs-apache/nginx-arch.png" alt="NGINX Architecture" />

### Performance comparison ###

Given that Apache needs to spawn a new thread for every connection, we can predict that Apache will take up more memory as the number of connections increase.  This is because each new thread gets allocated a block of memory by the OS.  Since NGINX runs on a single thread, the memory should stay more or less constant, though increase slowly as a small amount of memory is needed for each new connection.

Another useful metric is how many requests per second each server can handle.  Here, it's not immediately obvious who would win.  However, one useful bit of information is that context switching between threads on the CPU can be expensive.  That is, every time the CPU switches from processing one thread to processing another, there is some delay.  

If we imagine Apache opening a thousand threads for a thousand connections, a not insignificant chunk of the CPU's time would be consumed with context switching.  NGINX, on the other hand, would suffer zero overhead.

Performance benchmarks do in fact bear these conclusions out:

<img src="/images/posts/nginx-vs-apache/nginx-vs-apache-memory.png" alt="NGINX vs. Apache: Memory" />

<img src="/images/posts/nginx-vs-apache/nginx-vs-apache-rps.png" alt="NGINX vs. Apache: Requests per Second" width="715"/>

([Source](https://help.dreamhost.com/hc/en-us/articles/215945987-Web-server-performance-comparison))


### Conclusion ###

Clearly, on memory and performance, NGINX is the superior choice!  

One would then ask why anyone would continue to use Apache?  In my opinion, Apache is a holdover from an older period when NGINX was not around.  Also, for certain setups, it is easier to configure.  For example, a Python web server running Apache + mod_wsgi  is easier to setup than NGINX + Gunicorn (though in my opinion, not by much).

More importantly, unless you are dealing with hundreds of requests a second or memory is an issue, the choice should be up to personal preference.

