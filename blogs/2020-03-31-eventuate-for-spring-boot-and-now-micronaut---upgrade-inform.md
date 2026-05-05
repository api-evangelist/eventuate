---
title: "Eventuate for Spring Boot and now Micronaut - upgrade information"
url: "http://eventuate.io/post/eventuate/2020/03/31/upcoming-eventuate-release.html"
date: "Tue, 31 Mar 2020 09:01:00 +0000"
author: ""
feed_url: "https://eventuate.io/feed"
---
<h1 id="upgrading-to-the-upcoming-release-of-eventuate">Upgrading to the upcoming release of Eventuate</h1>

<p>Eventuate is a platform that solves the distributed data management problems inherent in a microservice architecture.
It makes it straightforward to use <a href="/post/eventuate/2020/02/24/why-eventuate.html">patterns such as Sagas, CQRS and Event Sourcing</a> and enables you focus on your business logic.</p>

<p>The latest versions of Eventuate Local (0.32.0.RC4), Eventuate Tram (0.24.0.RC4), Eventuate Tram Sagas (0.13.0.RC4) now support both Spring Boot and Micronaut.
We plan to make .RELEASE versions shortly.</p>

<p>Here are the Eventuate Micronaut example applications:</p>

<ul>
  <li>
    <p><a href="https://github.com/eventuate-tram-examples/eventuate-tram-examples-micronaut-customers-and-orders">Eventuate Tram Customers and Orders - Micronaut</a> - demonstrates how to maintain data consistency in an Micronaut, JPA-based microservice architecture using <a href="http://microservices.io/patterns/data/saga.html">choreography-based sagas</a>.</p>
  </li>
  <li>
    <p><a href="https://github.com/eventuate-tram-examples/eventuate-tram-sagas-micronaut-examples-customers-and-orders">Eventuate Tram Sagas Customers and Orders - Micronaut</a> - demonstrates how to maintain data consistency in an Micronaut, JPA-based microservice architecture using <a href="http://microservices.io/patterns/data/saga.html">orchestration-based sagas</a>.</p>
  </li>
  <li>
    <p><a href="https://github.com/eventuate-examples/eventuate-micronaut-examples-customers-and-orders">Eventuate Customers and Orders - Micronaut</a> - demonstrates how to maintain data consistency in an Micronaut, JPA-based microservice architecture using Micronaut, Event Sourcing, <a href="http://microservices.io/patterns/data/saga.html">choreography-based sagas</a> and CQRS.</p>
  </li>
</ul>

<p>There are <a href="/exampleapps.html">numerous other example applications for Spring Boot</a>.</p>

<p>Please note that adding support for Micronaut required changing the names of both Maven artifacts and packages.
In order to upgrade to these versions, you must edit your source code.
To simplify the process, we created a <a href="https://github.com/eventuate-tram/eventuate-upgrade-scripts">Python-based upgrade script</a> that makes all but one of the needed renames for Eventuate Local, Tram and Saga applications.
It would be great if you try it and provide feedback.</p>
