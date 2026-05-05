---
title: "New Eventuate platform APIs that support Async API: part 1 - events"
url: "http://eventuate.io/post/eventuate/2025/03/11/eventuate-new-apis-part-1.html"
date: "Tue, 11 Mar 2025 08:01:00 +0000"
author: ""
feed_url: "https://eventuate.io/feed"
---
<h1 id="new-eventuate-platform-apis-that-support-async-api-part-1---events">New Eventuate platform APIs that support Async API: part 1 - events</h1>

<p>I recently developed a <a href="https://github.com/eventuate-platform/eventuate-tram-spring-wolf-support/">plugin for SpringWolf</a> that configures an Eventuate service to expose an <code class="language-plaintext highlighter-rouge">/springwolf/docs</code> endpoint that returns an Async API document describing the messages that the service sends and receives.
While implementing the plugin, however, I discovered that the existing Eventuate APIs were not well suited to exposing Async API-style metadata.
For example, the messages sent by the service — and the channels to which it sent them — were not explicitly defined; instead, these details were hidden within the code</p>

<p>This realization prompted the creation of a new set of Eventuate APIs for sending and receiving messages.
The old APIs still work but only limited metadata can be generated from them.
In this article, I’ll describe the new APIs for publishing and subscribing to events.
A later article will describe the new APIs for commands and sagas.
Let’s first look at the changes to event publishing.</p>

<h2 id="event-publishing">Event publishing</h2>

<p>Let’s first look at the old way of publishing events.
After that I describe the new approach.</p>

<h3 id="original-event-publishing-api">Original event publishing API</h3>

<p>Previously, domain logic simply used the <code class="language-plaintext highlighter-rouge">DomainEventPublisher</code> to publish events:</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">CustomerService</span> <span class="o">{</span>

  <span class="kd">private</span> <span class="nc">DomainEventPublisher</span> <span class="n">domainEventPublisher</span><span class="o">;</span>

  <span class="nd">@Transactional</span>
  <span class="kd">public</span> <span class="nc">Customer</span> <span class="nf">createCustomer</span><span class="o">(</span><span class="nc">String</span> <span class="n">name</span><span class="o">,</span> <span class="nc">Money</span> <span class="n">creditLimit</span><span class="o">)</span> <span class="o">{</span>
    <span class="o">...</span>
    <span class="n">domainEventPublisher</span><span class="o">.</span><span class="na">publish</span><span class="o">(</span><span class="nc">Customer</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="n">customer</span><span class="o">.</span><span class="na">getId</span><span class="o">(),</span> <span class="n">customerWithEvents</span><span class="o">.</span><span class="na">events</span><span class="o">);</span>
    <span class="k">return</span> <span class="n">customer</span><span class="o">;</span>
  <span class="o">}</span>
</code></pre></div></div>

<p>As a result, the channels and event types were not explicitly defined.</p>

<h3 id="new-event-publishing-api">New event publishing API</h3>

<p>The new approach consists of defining one or more <code class="language-plaintext highlighter-rouge">DomainEventPublisherForAggregate</code> <code class="language-plaintext highlighter-rouge">@Beans</code>.
A <code class="language-plaintext highlighter-rouge">DomainEventPublisherForAggregate</code> publishes events for a specific aggregate.
Although defining <code class="language-plaintext highlighter-rouge">DomainEventPublisherForAggregates</code> involves more code, it explicitly defines the channels and events.
The Eventuate plugin for Spring Wolf uses the <code class="language-plaintext highlighter-rouge">DomainEventPublisherForAggregate</code> <code class="language-plaintext highlighter-rouge">@Beans</code> to generate an Async API document describing the published events and their channels.</p>

<p>Here’s the definition of a <code class="language-plaintext highlighter-rouge">DomainEventPublisherForAggregate</code> for the <code class="language-plaintext highlighter-rouge">Customer</code> aggregate:</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">public</span> <span class="kd">interface</span> <span class="nc">CustomerEventPublisher</span> <span class="kd">extends</span> <span class="nc">DomainEventPublisherForAggregate</span><span class="o">&lt;</span><span class="nc">Customer</span><span class="o">,</span> <span class="nc">Long</span><span class="o">,</span> <span class="nc">CustomerEvent</span><span class="o">&gt;</span> <span class="o">{</span>
<span class="o">}</span>
</code></pre></div></div>

<p>It’s injected into the <code class="language-plaintext highlighter-rouge">CustomerService</code>:</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">CustomerService</span> <span class="o">{</span>

  <span class="kd">private</span> <span class="kd">final</span> <span class="nc">CustomerEventPublisher</span> <span class="n">customerEventPublisher</span><span class="o">;</span>

  <span class="kd">public</span> <span class="kt">void</span> <span class="nf">reserveCredit</span><span class="o">(</span><span class="kt">long</span> <span class="n">orderId</span><span class="o">,</span> <span class="kt">long</span> <span class="n">customerId</span><span class="o">,</span> <span class="nc">Money</span> <span class="n">orderTotal</span><span class="o">)</span> <span class="o">{</span>

      <span class="nc">CustomerCreditReservedEvent</span> <span class="n">customerCreditReservedEvent</span> <span class="o">=</span>
              <span class="k">new</span> <span class="nf">CustomerCreditReservedEvent</span><span class="o">(</span><span class="n">customerId</span><span class="o">,</span> <span class="n">orderId</span><span class="o">);</span>

      <span class="n">customerEventPublisher</span><span class="o">.</span><span class="na">publish</span><span class="o">(</span><span class="n">customer</span><span class="o">,</span> <span class="n">customerCreditReservedEvent</span><span class="o">);</span>


</code></pre></div></div>

<p>Finally, here is the <code class="language-plaintext highlighter-rouge">@Bean</code> implementation of the <code class="language-plaintext highlighter-rouge">CustomerEventPublisher</code>:</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nd">@Component</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">CustomerEventPublisherImpl</span> <span class="kd">extends</span> <span class="nc">AbstractDomainEventPublisherForAggregateImpl</span><span class="o">&lt;</span><span class="nc">Customer</span><span class="o">,</span> <span class="nc">Long</span><span class="o">,</span> <span class="nc">CustomerEvent</span><span class="o">&gt;</span> <span class="kd">implements</span> <span class="nc">CustomerEventPublisher</span> <span class="o">{</span>

    <span class="kd">public</span> <span class="nf">CustomerEventPublisherImpl</span><span class="o">(</span><span class="nc">DomainEventPublisher</span> <span class="n">domainEventPublisher</span><span class="o">)</span> <span class="o">{</span>
        <span class="kd">super</span><span class="o">(</span><span class="nc">Customer</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="nl">Customer:</span><span class="o">:</span><span class="n">getId</span><span class="o">,</span> <span class="n">domainEventPublisher</span><span class="o">,</span> <span class="nc">CustomerEvent</span><span class="o">.</span><span class="na">class</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre></div></div>

<p>Let’s now look at the new event handling API.</p>

<h2 id="subscribing-to-events">Subscribing to events</h2>

<p>Let’s first look at the old way of subscribing to events.
After that I describe the new approach.</p>

<h3 id="original-event-handling-api">Original event handling API</h3>

<p>Previously, event handlers were configured by defining a class that constructed <code class="language-plaintext highlighter-rouge">DomainEventHandlers</code>:</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderEventConsumer</span> <span class="o">{</span>

  <span class="kd">public</span> <span class="nc">DomainEventHandlers</span> <span class="nf">domainEventHandlers</span><span class="o">()</span> <span class="o">{</span>
    <span class="k">return</span> <span class="nc">DomainEventHandlersBuilder</span>
            <span class="o">.</span><span class="na">forAggregateType</span><span class="o">(</span><span class="s">"io.eventuate.examples.tram.ordersandcustomers.orders.domain.Order"</span><span class="o">)</span>
            <span class="o">.</span><span class="na">onEvent</span><span class="o">(</span><span class="nc">OrderCreatedEvent</span><span class="o">.</span><span class="na">class</span><span class="o">,</span> <span class="k">this</span><span class="o">::</span><span class="n">handleOrderCreatedEvent</span><span class="o">)</span>
            <span class="o">...</span>
            <span class="o">.</span><span class="na">build</span><span class="o">();</span>
  <span class="o">}</span>

  <span class="kd">public</span> <span class="kt">void</span> <span class="nf">handleOrderCreatedEvent</span><span class="o">(</span><span class="nc">DomainEventEnvelope</span><span class="o">&lt;</span><span class="nc">OrderCreatedEvent</span><span class="o">&gt;</span> <span class="n">domainEventEnvelope</span><span class="o">)</span> <span class="o">{</span>
    <span class="o">...</span>
</code></pre></div></div>

<p>The <code class="language-plaintext highlighter-rouge">DomainEventHandlers</code> was then passed to a <code class="language-plaintext highlighter-rouge">DomainEventDispatcherFactory</code>:</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">CustomerConfiguration</span> <span class="o">{</span>

  <span class="nd">@Bean</span>
  <span class="kd">public</span> <span class="nc">OrderEventConsumer</span> <span class="nf">orderEventConsumer</span><span class="o">(</span><span class="nc">CustomerService</span> <span class="n">customerService</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nf">OrderEventConsumer</span><span class="o">(</span><span class="n">customerService</span><span class="o">);</span>
  <span class="o">}</span>

  <span class="nd">@Bean</span>
  <span class="kd">public</span> <span class="nc">DomainEventDispatcher</span> <span class="nf">domainEventDispatcher</span><span class="o">(</span><span class="nc">OrderEventConsumer</span> <span class="n">orderEventConsumer</span><span class="o">,</span> <span class="nc">DomainEventDispatcherFactory</span> <span class="n">domainEventDispatcherFactory</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">return</span> <span class="n">domainEventDispatcherFactory</span><span class="o">.</span><span class="na">make</span><span class="o">(</span><span class="s">"orderServiceEvents"</span><span class="o">,</span> <span class="n">orderEventConsumer</span><span class="o">.</span><span class="na">domainEventHandlers</span><span class="o">());</span>
  <span class="o">}</span>
</code></pre></div></div>

<p>The Eventuate plugin for Spring Wolf can use the <code class="language-plaintext highlighter-rouge">DomainEventDispatcher</code> <code class="language-plaintext highlighter-rouge">@Beans</code> to generate an Async API document describing the subscribed to events and their channels.
However, there’s no obvious way to customize the Async API document by, for example, specifying additional documentation.</p>

<h3 id="new-event-handling-api">New event handling API</h3>

<p>The new approach is to define beans that have methods annotated with <code class="language-plaintext highlighter-rouge">@EventuateDomainEventHandler</code>.
The annotation specifies the event handler’s subscriber ID and the channel.
The event type is obtained from the method parameter.</p>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nd">@Component</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderEventConsumer</span> <span class="o">{</span>
  <span class="kd">private</span> <span class="nc">Logger</span> <span class="n">logger</span> <span class="o">=</span> <span class="nc">LoggerFactory</span><span class="o">.</span><span class="na">getLogger</span><span class="o">(</span><span class="n">getClass</span><span class="o">());</span>

  <span class="kd">private</span> <span class="nc">CustomerService</span> <span class="n">customerService</span><span class="o">;</span>

  <span class="kd">public</span> <span class="nf">OrderEventConsumer</span><span class="o">(</span><span class="nc">CustomerService</span> <span class="n">customerService</span><span class="o">)</span> <span class="o">{</span>
    <span class="k">this</span><span class="o">.</span><span class="na">customerService</span> <span class="o">=</span> <span class="n">customerService</span><span class="o">;</span>
  <span class="o">}</span>

  <span class="nd">@EventuateDomainEventHandler</span><span class="o">(</span><span class="n">subscriberId</span> <span class="o">=</span> <span class="s">"OrderEventConsumer"</span><span class="o">,</span> <span class="n">channel</span> <span class="o">=</span> <span class="s">"io.eventuate.examples.tram.ordersandcustomers.orders.domain.Order"</span><span class="o">)</span>
  <span class="kd">public</span> <span class="kt">void</span> <span class="nf">handleOrderCreatedEvent</span><span class="o">(</span><span class="nc">DomainEventEnvelope</span><span class="o">&lt;</span><span class="nc">OrderCreatedEvent</span><span class="o">&gt;</span> <span class="n">domainEventEnvelope</span><span class="o">)</span> <span class="o">{</span>
    <span class="nc">OrderCreatedEvent</span> <span class="n">event</span> <span class="o">=</span> <span class="n">domainEventEnvelope</span><span class="o">.</span><span class="na">getEvent</span><span class="o">();</span>
    <span class="n">customerService</span><span class="o">.</span><span class="na">reserveCredit</span><span class="o">(</span><span class="nc">Long</span><span class="o">.</span><span class="na">parseLong</span><span class="o">(</span><span class="n">domainEventEnvelope</span><span class="o">.</span><span class="na">getAggregateId</span><span class="o">()),</span>
            <span class="n">event</span><span class="o">.</span><span class="na">orderDetails</span><span class="o">().</span><span class="na">customerId</span><span class="o">(),</span> <span class="n">event</span><span class="o">.</span><span class="na">orderDetails</span><span class="o">().</span><span class="na">orderTotal</span><span class="o">());</span>
  <span class="o">}</span>
</code></pre></div></div>

<p>The Eventuate plugin for Spring Wolf can search the application context for <code class="language-plaintext highlighter-rouge">@Beans</code> that have methods annotated with <code class="language-plaintext highlighter-rouge">@EventuateDomainEventHandler</code>.
Moreover, in the future, it will support additional annotations that allow you to customize the generated Async API document.</p>

<h2 id="show-me-the-code">Show me the code</h2>

<ul>
  <li>The Eventuate Spring Wolf repository contains some <a href="https://github.com/eventuate-platform/eventuate-tram-spring-wolf-support?tab=readme-ov-file#example-async-api-documentation">example Async API docs</a></li>
  <li>The <a href="https://github.com/eventuate-tram/eventuate-tram-examples-customers-and-orders/tree/development">development branch</a> of the <code class="language-plaintext highlighter-rouge">eventuate-tram-examples-customers-and-orders</code> repository contains an example application that uses the new APIs.</li>
</ul>

<h2 id="whats-next">What’s next</h2>

<p>A later post will describe the new APIs for sagas and command handlers.</p>
