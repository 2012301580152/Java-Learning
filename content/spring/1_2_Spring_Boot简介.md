# 1.2 Spring Boot 简介

    @RequestMapping(path="/design",                      // <1>
                produces="application/json")
    @CrossOrigin(origins="*")        // <2>
# PART1 FOUNDATIONAL SPRING
## Getting started with Spring
## Developing web applications
## Working with data
# PART2 INTEGRATED SPRING
## 6 Creating REST service
### Enabling hypermedia

Hardcoding API URLs and using string manipulation on API client code makes the client code brittle. Hypermedia as the Engine of Application State, or HATEOAS, is means of creating self-describing APIs wherein resource returned from an API contain links to related resource. To enable hypermedia in the Application API, you'll need to add the spring HATEOAS starter dependency to the build:
    
    spring-boot-starter-hateoas


### Enabling data-backed services

To start using Spring Data REST, artu add the following dependency to your build:

    spring-boot-starter-data-restart

## 7 Consuming REST service
art
art API with
art
* RestTemplate-
* Traversion-A hyperlink-aware, synchronous REST client provided by Spring HATEOAS. Inspired from a JavaScript library of the same name.
* WebClient

### 

### Navigating REST APIs with Traversion

## 8 Sending messages asynchronously

### Sending messages with JMS

## 9 Integrating Spring

# PART3 REACTIVE SPRING

## Introducing Reactor
As we develop application code, there are two style of code we can write: imperative and reactive:

* Imperative code is a lot like that absurd hypothetical newspaper subscription. It's a serial set of tasks, each running one at a time, each after the previous task. Date is processed in bulk and can't be handed over to the next task until the previous task has completed its work on the bulk of data.
* Reactive code is a lot like a real newspaper subscription. A set of tasks is defined to process data, but those tasks can run in parallel. Each task can process subsets of the data, handing it off to the next task in line while it continues to work on another subset of the data.



# PART4 CLOUD-NATIVE SPRING

# PART5 DEPLOYED SPRING
