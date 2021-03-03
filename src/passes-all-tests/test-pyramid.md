## Test pyramid
There are different kinds of tests, each one meant for a different purpose. Before describing the most common ones, let's
clarify in advance a couple of terms. For _service_ we mean a bunch of code deployed as a whole. For
_external dependencies_ we mean anything that is reached over the network, like a database, another team REST endpoint, a queue, third party apis, etc.

* **Acceptance tests**  
  Check that a feature behaves as expected across all service layers (e.g. back end and front end) or even
  across different services. External dependencies are replaced through libraries like [LocalStack](https://github.com/localstack/localstack)
  or [Wiremock](http://wiremock.org/). In a web application for instance, acceptance tests are defined with tools like [Selenium](https://www.selenium.dev/)
  or [Cypress](https://www.cypress.io/).
* **Functional tests**  
  Check that a feature behaves as expected considering a single layer of a service. For example, if a service have both back end and
  front end, there will be distinct functional tests for the back end and front end. External dependencies are replaced either
  by libraries like [LocalStack](https://github.com/localstack/localstack) or by [doubles](test-doubles.html).
  Functional tests are also called component tests.
* **Integration tests**  
  Check that a service integrates correctly with external dependencies. External dependencies are replaced by libraries like
  [LocalStack](https://github.com/localstack/localstack) or [Wiremock](http://wiremock.org/). If you use code [doubles](test-doubles.html)
  for the external dependencies then it is a unit test.
* **Unit tests**  
  Check that code inside one class behaves as expected. Unit tests are most valuable when testing business logic: if a class is just a
  [delegator](https://en.wikipedia.org/wiki/Delegation_pattern) or just coordinates other classes, do not use unit tests
  as functional tests already provide coverage. If the class under test uses other classes whose construction is cumbersome,
  those can be replaced with   [doubles](test_doubles.html)


The above list is ordered by how much time a test takes to execute, from the slowest (acceptance) to the fastest (unit).
For this reason, it is recommended to have a pyramid of tests: a handful of acceptance tests, some functional and integration
tests and many unit tests. In particular:

* **Acceptance tests**  
  Only for default uses of a feature
* **Functional tests**  
  For both default and exceptional-erroneous uses of a feature
* **Integration tests**  
  For both default and exceptional-erroneous integrations with external dependencies
* **Unit tests**  
  For both default and exceptional-erroneous usage of a single class

<br/>  

#### Recommended reads
* [Test Pyramid (in short) - Martin Fowler](https://martinfowler.com/bliki/TestPyramid.html)  
* [Test Pyramid (in depth) - Ham Vocke](https://martinfowler.com/articles/practical-test-pyramid.html)  
* [Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)
* [Fixing a Test Hourglass, Google testing blog - Alan Myrvold](https://testing.googleblog.com/2020/11/fixing-test-hourglass.html)