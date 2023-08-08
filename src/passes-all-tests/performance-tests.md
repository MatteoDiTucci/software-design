## Performance tests

Performance tests verify how a system behaves in a production like environment. 
We are particularly interested in metrics that affect how the users perceive the system from the outside, like for instance 
response time or throughput. However, performance tests are also useful to verify the cost efficiency of a system,
for instance verifying that hardware resources are not underutilised.

Let's see the main aspects to consider when dealing with performance tests.

### Vocabulary
Performance, load and stress tests differ by the type of load used for the test.
* **Performance tests**: expected load which represents the daily baseline for a system
* **Load tests**: expected peaks of load which are recurrently handled by the system
* **Stress tests**: unusual peaks of load, so big that we expect failures to happen

### Target state
Before starting a performance test, it is important to specify which metrics we care about. For instance, response time,
number of errors, throughput, etc.
This is important because it helps us understand how to shape the load of the test and what to measure.

### Current state
Performance tests guide our efforts to improve the performance of a system. Before starting any performance related refactoring,
we should run performance tests to measure the current value of the metrics we are interested in. After every refactoring,
by running again the same performance tests, we measure the magnitude of the improvement and how much is still left to do.

### Theory of constraints
When performance tests highlight a low performing metric, it is important to identify what is its main contributor. 
For instance, if response time is low, we need to identify how the time is spent across the system at different granularity.
By this, we mean not only narrowing down macro components like front-end or database, but also architectural layers like
domain or presentation, up to class of function level if necessary.

We should spend all our efforts on the single main contributor of the low performing metric. Once solved, move to the new main contributor.

### Continuous testing
Performance tests should happen continuously, to immediately catch any degradation introduced by new code revisions. 
For this reason, a great approach is to have performance tests inside the continuous deployment pipeline. If not possible, 
we should at least have a dedicated pipeline that runs performance tests once a day.

### Test in production
Performance tests should run on environments as close as possible to production in terms of infrastructure, data, load, 
third-party integrations, etc. As it is costly to build and maintain an exact copy of production, the best is to run performance
tests directly in production. To avoid affecting real customers, we can increase the performance test load gradually and 
have test users in place.

### Observability
As it is impossible to anticipate every production issue, it is important to have observable systems. In terms of tooling
this translates to logging, monitoring and tracing. Once we are able to spot and fix production performance issues, we can later 
replicate their triggering conditions inside our performance tests so that they behave as a regression safety net. 


<br/>  

#### Recommended reads
* [Release It!: Design and Deploy Production-Ready Software (2nd edition) - Michael T. Nygard](https://www.goodreads.com/book/show/1069827.Release_It_)
* [Performance testing in a nutshell - Snrinivas Murty](https://www.thoughtworks.com/en-in/insights/blog/performance-testing-nutshell)
* [How NOT to Measure Latency - Gil Tene](https://www.infoq.com/presentations/latency-response-time/)
* [Constraint theory - Wikipedia](https://en.wikipedia.org/wiki/Theory_of_constraints)
* [Fixing Performance Regressions Before they Happen - Angus Croll, Netflix](https://netflixtechblog.com/fixing-performance-regressions-before-they-happen-eab2602b86fe)
* [Service level objective chapter of SRE book - Google](https://sre.google/sre-book/monitoring-distributed-systems/)
* [How they load test - Aliaksandr Belik](https://github.com/aliesbelik/how-they-load)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.