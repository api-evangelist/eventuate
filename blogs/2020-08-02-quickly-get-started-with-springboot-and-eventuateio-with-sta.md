---
title: "Quickly get started with @SpringBoot and @EventuateIO with start.eventuate.io"
url: "http://eventuate.io/post/eventuate/2020/08/02/getting-started-with-start-eventuate-io.html"
date: "Sun, 02 Aug 2020 09:01:00 +0000"
author: ""
feed_url: "https://eventuate.io/feed"
---
<h1 id="quickly-get-started-with-springboot-and-eventuateio-with-starteventuateio">Quickly get started with @SpringBoot and @EventuateIO with start.eventuate.io</h1>

<p>The website <a href="https://start.eventuate.io/">start.eventuate.io</a> is a quick and convenient way to start the development of an Eventuate/Spring Boot-based service.</p>

<p>There are many Eventuate dependencies to choose from:</p>

<ul>
  <li>
    <p>If you want to using event sourcing, then select the Eventuate Local dependencies.</p>
  </li>
  <li>
    <p>Otherwise, if you want to use traditional persistence then select the Eventuate Tram and Eventuate Saga dependencies.
Don’t forget you need to pick exactly one transport for the message broker that you want to use.</p>
  </li>
</ul>

<p><a href="https://start.eventuate.io/">Start.eventuate.io</a> built on <a href="https://github.com/spring-io/initializr/">Spring Initialzr</a>.
You can, for example, use the command line to generate a service project:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>curl https://start.eventuate.io/starter.zip \
-d type=maven-project \
-d language=java \
-d platformVersion=2.3.2.RELEASE \
-d packaging=jar \
-d jvmVersion=11 \
-d groupId=io.eventuate.example.sagas \
-d artifactId=orchestrator \
-d name=orchestrator \
-d description=Demo%20project%20Eventuate%2C%20domain%20events%2C%20and%20saga%20orchestration \
-d packageName=io.eventuate.example.sagas.orchestrator \
-d dependencies=eventuatetramdomainevents,eventuatetramsagaorchestrator,eventuatetramkafka,actuator \
-o demo.zip
</code></pre></div></div>

<p><img class="img-responsive" src="/i/Start.Eventuate.IO.smaller.png" /></p>
