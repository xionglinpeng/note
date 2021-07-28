# EventBus





Dispatches events to listeners, and provides ways for listeners to register themselves.

The EventBus allows publish-subscribe-style communication between components without requiring the components to explicitly register with one another (and thus be aware of each other). It is designed exclusively to replace traditional Java in-process event distribution using explicit registration. It is not a general-purpose publish-subscribe system, nor is it intended for interprocess communication.

**Receiving Events**

To receive events, an object should:

1. Expose a public method, known as the event subscriber, which accepts a single argument of the type of event desired;

2. Mark it with a Subscribe annotation;

3. Pass itself to an EventBus instance's register(Object) method.


**Posting Events**

To post an event, simply provide the event object to the post(Object) method. The EventBus instance will determine the type of event and route it to all registered listeners.

Events are routed based on their type — an event will be delivered to any subscriber for any type to which the event is assignable. This includes implemented interfaces, all superclasses, and all interfaces implemented by superclasses.

When post is called, all registered subscribers for an event are run in sequence, so subscribers should be reasonably quick. If an event may trigger an extended process (such as a database load), spawn a thread or queue it for later. (For a convenient way to do this, use an AsyncEventBus.)

**Subscriber Methods**

Event subscriber methods must accept only one argument: the event.

Subscribers should not, in general, throw. If they do, the EventBus will catch and log the exception. This is rarely the right solution for error handling and should not be relied upon; it is intended solely to help find problems during development.

The EventBus guarantees that it will not call a subscriber method from multiple threads simultaneously, unless the method explicitly allows it by bearing the AllowConcurrentEvents annotation. If this annotation is not present, subscriber methods need not worry about being reentrant, unless also called from outside the EventBus.

**Dead Events**

If an event is posted, but no registered subscribers can accept it, it is considered "dead." To give the system a second chance to handle dead events, they are wrapped in an instance of DeadEvent and reposted.

If a subscriber for a supertype of all events (such as Object) is registered, no event will ever be considered dead, and no DeadEvents will be generated. Accordingly, while DeadEvent extends Object, a subscriber registered to receive any Object will never receive a DeadEvent.

This class is safe for concurrent use.

See the Guava User Guide article on [EventBus](https://github.com/google/guava/wiki/EventBusExplained).





## 1、How to create the EventBus

## 2、Subscribing to events

## 3、Posting the events

## 4、Event Handler

## 5、The AsyncEventBus

## 6、EventBus Principles and design

