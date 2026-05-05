---
title: "Eventuate explained using microservices patterns"
url: "http://eventuate.io/post/eventuate/2020/02/24/why-eventuate.html"
date: "Mon, 24 Feb 2020 09:01:00 +0000"
author: ""
feed_url: "https://eventuate.io/feed"
---
<h1 id="eventuate-explained-using-microservices-patterns">Eventuate explained using microservices patterns</h1>

<p>A while back I came up with this diagram that explains Eventuate using microservice architecture patterns.</p>

<p><img class="img-responsive" src="/i/Why_Eventuate.png" /></p>

<p>Let’s look at each of the patterns in turn starting with the Database per Service pattern.</p>

<h2 id="database-per-service-pattern">Database per Service pattern</h2>

<p>A key characteristic of the Microservice Architecture is loose coupling.
There are two types of coupling:</p>

<ul>
  <li>design-time coupling - changing one service requires other services to change.
Services must be loosely coupled from a design-time perspective in order to ensure that teams are loosely coupled and, therefore, productive.</li>
  <li>runtime coupling - the failure of one service prevents another service from handling a request.
Services must also be loosely coupled from a runtime perspective in order to maximize availability.</li>
</ul>

<p>This requirement for loose coupling significantly impacts the database architecture and creates the need for the <a href="https://microservices.io/patterns/data/database-per-service.html">Database per Service pattern</a>.
At a minimum, to avoid design time coupling, services must not share database tables.
And, <em>ideally</em>, to reduce runtime coupling, each service should have its own database server.</p>

<h2 id="distributed-data-management-patterns">Distributed data management patterns</h2>

<p>The <a href="https://microservices.io/patterns/data/database-per-service.html">Database per Service pattern</a> creates distributed data management challenges.
There are two key distributed data management patterns that solve these problems:</p>

<ul>
  <li>Saga pattern - implements transactions that span services</li>
  <li>CQRS pattern - implements queries that span services</li>
</ul>

<h3 id="transaction-management-with-the-saga-pattern">Transaction management with the Saga pattern</h3>

<p>Some update operations (a.k.a. <a href="https://microservices.io/articles/glossary#command">commands</a>) need to update data in multiple services.
Such an operation typically doesn’t use traditional distributed transactions (2PC) since that’s a type of runtime coupling.
Instead, a command that spans services must be implemented using the <a href="https://microservices.io/patterns/data/saga.html">Saga pattern</a>.
A saga is a series of local transactions that are coordinated by the participating services exchanging <a href="https://microservices.io/patterns/communication-style/messaging.html">messages</a>.</p>

<p>As I describe in this <a href="http://chrisrichardson.net/post/antipatterns/2019/07/09/developing-sagas-part-1.html">series of blog posts about sagas</a> there are two types of saga coordination mechanisms:</p>

<ul>
  <li>choreography-based sagas - services collaborate by exchanging <a href="https://microservices.io/patterns/data/domain-event.html">domain events</a></li>
  <li>orchestration-based sagas - a centralized coordinator sending command messages to participants, which respond with reply messages</li>
</ul>

<h3 id="implementing-queries-with-the-cqrs-pattern">Implementing queries with the CQRS pattern</h3>

<p>Some <a href="https://microservices.io/articles/glossary#query">queries</a> need to retrieve data from multiple services.
Since each database is private, the data can only be retrieved using each service’s API.
One option is to use <a href="https://microservices.io/patterns/data/api-composition.html">API Composition</a> and simply invokes query operations on the services.
However, sometimes API composition is inefficient and instead the application must use the <a href="https://microservices.io/patterns/data/cqrs.html">CQRS pattern</a>.</p>

<p>The CQRS pattern maintains an easily queryable replica of the data using <a href="https://microservices.io/patterns/data/domain-event.html">domain events</a> published by the services that own the data.</p>

<h2 id="how-eventuate-solves-these-distributed-data-management-challenges">How Eventuate solves these distributed data management challenges</h2>

<p>Eventuate is a family of frameworks that solve the distributed data management challenges in microservice architecture.
By using Eventuate, you can easily implement the Saga and CQRS patterns in your application.
Eventuate also provides the messaging primitives for general purpose <em>transactional</em> messaging between services.</p>

<p>The following diagram shows the Eventuate family of frameworks:</p>

<p><img class="img-responsive" src="/i/Eventuate_Frameworks.png" /></p>

<p>There are the following frameworks:</p>

<ul>
  <li>Eventuate Tram Sagas - an orchestration-based saga framework</li>
  <li>Eventuate Tram - a transactional message framework</li>
  <li>Eventuate Local - an event sourcing framework</li>
</ul>

<p>Let’s look at the capabilities of these frameworks.</p>

<h3 id="orchestration-based-sagas">Orchestration-based sagas</h3>

<p>The <a href="https://github.com/eventuate-tram/eventuate-tram-sagas">Eventuate Tram Sagas</a> framework implements orchestration-based sagas.
It provides an easy to use Java-based domain-specific language for the defining the steps of your saga and their compensating transactions.
The saga execution engine is embedded within your services, which simplifies the architecture and improves scalability and availability.</p>

<h3 id="choreography-based-sagas-and-cqrs">Choreography-based sagas and CQRS</h3>

<p>Eventuate makes it easy to implement event-driven microservices that use choreography-based sagas and CQRS.
It provides an API for publishing and consuming events.
Services can use Eventuate to publish events as part of a database transaction that updates business entity.
Other services can use Eventuate to consume events with automatic duplicate detection, which ensures that your <a href="https://microservices.io/patterns/communication-style/idempotent-consumer.html">message handlers are idempotent</a>.</p>

<p>Eventuate provides different event-driven programming models</p>

<ul>
  <li>Explicit event publishing - use for applications that persist business entities using traditional (e.g. JDBC/JPA) persistence.
The <a href="https://eventuate.io/abouteventuatetram.html">Eventuate Tram framework</a> provides APIs for publishing and consuming events.</li>
  <li><a href="https://microservices.io/patterns/data/event-sourcing.html">Event sourcing</a> - an event-centric way of writing your business logic and persisting your business objects. Event sourcing is useful for application that need to maintain the history of changes to business entities.
The <a href="https://eventuate.io/usingeventuate.html">Eventuate Local framework</a> is Eventuate’s event sourcing framework.</li>
</ul>

<h3 id="foundational-messaging-patterns">Foundational messaging patterns</h3>

<p>These higher level capabilities are built on several foundational patterns:</p>

<ul>
  <li>
    <p><a href="https://microservices.io/patterns/data/transactional-outbox.html">Transactional Outbox</a> - publish a message as part of the database transaction that updates business entities.
This pattern is essential for maintaining consistency.
Without it, there is a risk of either updating the database without sending a message or sending a message without the corresponding database update.</p>
  </li>
  <li>
    <p><a href="https://microservices.io/patterns/communication-style/idempotent-consumer.html">Idempotent Consumer</a> - detect and discard duplicate messages by tracking the messages that have already been processed.
This pattern handles the scenario where a failure causes the message broker to deliver a message more than once.</p>
  </li>
</ul>

<p>This patterns are implemented by both the Eventuate Tram and Eventuate Local frameworks.</p>

<h2 id="learn-more">Learn more</h2>

<p>To learn more please take a look at the following resources:</p>

<ul>
  <li>Read <a href="https://eventuate.io/gettingstarted.html">Eventuate Getting Started Guide</a></li>
  <li>Read <a href="https://microservices.io/book">Chris Richardson’s microservice patterns book</a></li>
</ul>
