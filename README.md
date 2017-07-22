# LFX
LFX framework for .NET platform

## Introduction

LFX is a lightweight framework for .NET platform written in C#. It is low-ceremony, simple but flexible and powerful library for building enterprise services. Built on .NET Standard library enables running on .NET Core and full framework. The emphasis is on brevity of user code, as smaller code is easier to maintain. Configuration by convention is main principle, removing lot of boilerplate code.

## Features

### Handlers

Both synchronous handlers (services) and asynchronous handlers (message queue handlers) are supported. Service handlers can be easily integrated with any REST framework (like Nancy) and have support for security, feature flags per user roles and support for system-wide rate limiting. MQ handlers support database and most popular message queue technologies. There is support for dynamic tracing based on users/roles/handlers as it enables more flexible real-time debugging. MQ messages can be autocreated on configured schedule.

### Container

Basis of library, enables injecting of all features into service and MQ handlers. Provides a way of handler to communicate with outside world. All features are injectable, this enables unit testing immediately, with mock objects shipped in a library for easier testing of user code. All features are configurable, so custom implementations can be injected. One example of injection is getting current local and UTC time, something that is usually injected in non-elegant way, or used statically in the application, resulting in impossibility to unit-test. Other examples are encryption support, configuration, logging, security etc.

### Configuration

Provides configuration through container. Supports watches for changes in underlying objects. Providers are pluggable, whether it is JSON file, services like Consul etc. All objects created by container can have properties set by container on creation time, so configuration and code thus remain separated.

### Logging

Simple and extensible built-in logging library, can be used instead of any other logging library. Built-in targets: console, files, Redis publish, database. Supports sync and async mode, batch writes, failover. Application log is separated from security log as they provide different role.

### Event bus

Volatile event bus (publish-subscribe) for communication between processes is used throughout the library for management of components and other purposes, when the need to communicate over the wire between different processes in same application. Redis is favorite option as used event bus.

### Notifications

General purpose extensible notification framework with push and pull models.

### ORM

Layer built on top of repositories, abstracts away both SQL and NoSQL approaches to provide unique capabilities like instrumentation and propagation of changes in objects to subscribers through event bus. Queries and aggregates are first-class citizens, what enables instrumentation (they have assigned names). If publish-subscribe is configured on a query or aggregate, collection will be updated through event bus so it will be kept up-to-date for benefit of low latency or even less network traffic (if number of reads is much bigger than number of writes). This resembles in-memory grids as graph of queries can be constructed in runtime and the result is fast in-memory calculation without the need to go for expensive database calls. Ultimate goal is that a handler doesn't have to create any custom cache, but rely on built-in, transparent caches inside ORM layer – then access can be optimized by different configuration (physical distribution of related handlers to same process with lot of cached objects etc.)
Memory repository is built-in for purpose of unit-testing. File-based JSON repository is also provided as simple service might not need full database. ORM repository is built on top of open-source ORM library.
Beside these, ORM layer provides transparent encryption, validation of objects, access security, automatic field generation, log table population (auditing). Supports multiple cache levels.

### HTTP client

HTTP clients in .NET world still create HTTP requests manually. The goal of this implementation is to have a tool to create configuration based on HTTP traffic data (HAR file) and call to a service than can be made in a single statement. The only job of a developer is then only to interpret successful responses (errors end up throwing HttpServiceException). Framework will then take care of the following (that becomes configuration and is configurable on-the-fly as it will be persisted in store):
*	Endpoint addresses are not needed, service and methods have meaningful names
*	Message format – HTTP method type, headers
*	HTTP connection will be reused if possible in a single session, transparent to user
*	Timeouts will be set up after collecting some statistics (usually it is hard to set this value to proper level, while we cannot wait forever)
*	Session handling (serialization of cookies in store) if needed for a service
*	Authentication is handled transparently
*	Token handling for OAUTH, token is also saved in store
*	Time-related behaviour is handled by framework – for rate-limited services
*	Custom HTTP codes mapped to different scenarios as they are not standardized and at different service they can have different meaning
*	Collecting of statistics about usage of a service – times, number of calls, bytes in request/response
*	Handing of errors – retrying or opening a circuit breaker if needed and then checking for status of a site periodically
*	Proxy policy if we want to control the use of proxies

### Error handling

As this field is usually neglected or done poorly, it has special status in LFX framework. Exceptions are consolidated to few classes, with clear distinction of transient and logical errors so that system can adapt differently (transient errors can be retried and will affect circuit breakers).
Framework is aware of statuses of HTTP services and DB backends so it won't invoke the handler if the service is unavailable or rate-limited. Policies for MQ invocation are configurable so that more logical behavior is result. Everything works together in a cooperative manner, while the developer has to choose between few relatively simple concepts.

### Profiling and instrumentation

Handlers spend time in CPU work or doing external calls – ORM calls and HTTP service calls. As everything is instrumented, it is easy to construct a view of performance of the whole system. Instrumentation information is collected, aggregated and saved to store for a view and analysis. Special code then can advise how to change configuration to optimize the system to get better performance for user's particular case. As LFX is used in VPS servers special care is taken care to detect possible resource malfunctions in virtual servers, like resource sharing etc.
MQ handlers have priorities, rate limiting and paralellization configuration for optimal use of resources, as some operations are always real-time, some are batch-type while some external resources can be rate-limited. Framework is aware of what can expect from particular handler type (obtained through instrumentation) so it can estimate the impact of execution of single message handler on overall system performance.

### Dependencies

LFX depends on fewest of external libraries as possible – Sigil for dynamic code generation, NPoco for ORM SQL DB integration. Everything else is plugged-in once the user decide which components to use. Choice of serialization library is left to the user, currently we recommend Jil as it is fastest JSON serialization library.

