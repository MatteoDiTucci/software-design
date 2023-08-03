## Test for production
Correct code which passes all tests can still fail once deployed in production.
This happens because tests do not account for failures.   
Example of failures are external apis being slow to respond or a spike of user requests saturating hardware resources.
To make our code resilient we need to embrace failure and mitigate its impact.   

Failure usually happen where our code accesses the network (e.g. database, external api, etc.).
The following approaches help to mitigate its impact:

* **Fail fast**  
  When an external dependency is down, it is better to present users with errors for some functionality than bringing down the whole application.
  Use timeouts and circuit breakers on client side.
* **Isolate failures**  
  Each integration point should have its own pool of resources (e.g. thread pool, connection pool, etc.)
  Use bulkheads.
* **Buffer loads**  
  Improve availability by absorbing load in buffers.
  Use queues and user notifications instead of synchronous responses.
* **Test error scenarios**  
  Use integration tests that simulates error cases like server timeout, connection error, slow responses, etc.


<br/>  

#### Recommended reads
* [Release It!: Design and Deploy Production-Ready Software (2nd edition) - Michael T. Nygard](https://www.goodreads.com/book/show/1069827.Release_It_)
* [Chaos Engineering - Wikipedia](https://en.wikipedia.org/wiki/Chaos_engineering)
* [It takes more than a Circuit Breaker to create a resilient application - Bilgin Ibryam](https://developers.redhat.com/blog/2017/05/16/it-takes-more-than-a-circuit-breaker-to-create-a-resilient-application)
* [Fault Tolerance in a High Volume, Distributed System - Ben Christensen](https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a)
* [Circuit breaker pattern - Microsoft](https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)
* [Bulkhead pattern - Microsoft](https://docs.microsoft.com/en-us/azure/architecture/patterns/bulkhead)
* [Fuzz testing - Gitlab](https://about.gitlab.com/topics/devsecops/what-is-fuzz-testing/)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.