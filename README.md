# SOAP-Webservices with Apache CXF & SpringBoot using JAX-WS RI & JAXB - Annotations only, absolutely no XML
[![Build Status](https://travis-ci.org/jonashackt/soap-spring-boot-cxf.svg?branch=master)](https://travis-ci.org/jonashackt/soap-spring-boot-cxf)
[![Coverage Status](https://coveralls.io/repos/jonashackt/soap-spring-boot-cxf/badge.svg)](https://coveralls.io/r/jonashackt/soap-spring-boot-cxf)

As Example SOAP-Service I did some research, but after all the well-known [Weather-Service] seemed to be the best Use-Case, although (or because?) it is used by nearly every tutorial. It is really hard to find free SOAP-Services on the web.

But i had to extend the Weather-Service a lot through out development - e.g. Custom Exceptions, more complex
Input-Requests and a little less methods, so i can show my findings better. The biggest change was to split it into a WSDL (as "just the SOAP-interface") and a bunch of XSDs that import each other. That should represent a more complex domain and although they do not contain that much definitions, i can show many related techniques much better, that appear commonly in real-world scenarios.

So this example-project is capable for bigger Use-Cases in Realworld-Scenarios with huge WSDLs and lots of imported XSDs, which again import tons of other XSDs. If you want, test it with your Service and i appreciate feedback :)

### General choices

In the project I tried to use some relevant technologies for getting SOAP-Services running, like:
* [Spring] with the aim to use absolutely no XML-Configuration (just Annotations)
* [Spring Boot], for easy "not care about Container" (cause it has an embedded [Tomcat]) and simple deployment - like Microservices (without the "micro" in the interface, since we are bound to SOAP...)
* One of the most relevant SOAP-Stack [Apache CXF] 3 as the Webservice-Stack to expose the SOAP-Webservices
* Oracle´s JAX-WS RI (Reference Implementation) with the [JAX-WS-commons project] as "the Standard" to define Webservices in Java
* [JAXB Java-XML-Binding] for working with XML
* JAX-WS Commons for Generating the Class-Files for JAXB, managed by the maven plugin [jaxws-maven-plugin]

I reached my aim to not use any XML-configuration, but it was harder than i thought... If you look on some detail, you´ll see what i mean.

### HowTo Use

Run "mvn clean install"-command at command-line, to ensure that all necessary Java-Classes & JAXB-Bindings are generated

Then, you could use Spring Boot with maven to expose your SOAP-Webservices
```sh
mvn spring-boot:run
```
or run the build .jar-File with
```sh
java -jar soap-spring-boot-cxf-0.0.5-SNAPSHOT.jar
```

### Testing

For testing end-to-end purposes I would recommend also getting [SOAP-UI], where you can check WSDL/XSD-compliance of the provided services very easily and you "see" your services.

But getting to know, how stuff is working, it´s often better to have a look at some tests. There should be a amount of test-cases, that show standard (JAX-WS with CXF) ways to test webservices, but also non-standard approaches to test some
UseCases i came across developing e.g. the custom SoapFaults on incorrect XML-messages.

### Facade-Mode

Sometimes, you are in need of a facade-mode, where your implementation doesn´t call real backends and only returns Dummy-Responses. E.g. when you want to protect your backends when load is getting to high for them (not for your server :), that is based on solid Spring-technology) or even if you want to build up a new environment, where your backends are not available right from the start. And you want this configurable, so you can react fast, when needed.

For this Scenario, Spring´s powerful but yet easy to use [Profile-Mechanism] will serve you well. In combination with using org.springframework.core.io.Resource to load your Dummy-Response-Files instead of Java´s NIO.2 (that could [fuck you up] because of classloader-differences in other environments than your local machine), your done with that task quite fast.


### Done´s
* No XML-configuration, also for undocumented CXF-details :)
* Readable Namespace-Prefixes
* Testcases with Apache CXF
* Custom SoapFault, when non-schmeme-compliant or syntactically incorrect XML is send to the service
* Tests with Raw HTTP-Client for Reaction on syntactically incorrect XML
* Custom Exception in Weather-WSDL/XSDs
* Example of Controller and Mappers, that map to and from an internal Domain-Model - for loose coupling between generated JAXB-Classes and Backends
* Facade-Mode, that only returns Dummy-Responses, if configured
* Logging-Framework for centralization of logging and message-creation, including chance to define individial logging-Ids
* Webservice-Method that returns a PDF-File (you can view the base64-encoded String inside the Webservice´ Response with a small Angular/Boot-App I wrote for that: [base64gular])
* PDF-Test with asserts of the PDF-contents via [Pdfbox]
* Deployment to [Heroku], with inspiration from my colleague´s [blogpost] - see it in action (maybe you have to wait a while, cause it´s just a free Heroku-Dyno) [here] - or call it via [SOAP-UI]


## Loganalysis with [ELK-Stack]

If you´re going some steps further into a more production-ready environment, you´ll need a more indepth view what´s going on with your SOAP-Infrastructure. I used the [ELK-Stack] with Logstash -> Elasticsearch -> Kibana. I used the [logstash-logback-encoder] for getting JSONized Logoutputs directly into logstash´s input-phase.

![Kibana SOAP-Message Analytics](https://github.com/jonashackt/soap-spring-boot-cxf/blob/master/kibana_SOAP-Message-Analytics.png)

Making your SpringBoot-App ready for logstash, you have to add a maven-dependency and a logback.xml-File with the apropriate configuration, also described in [logstash-logback-encoder]. Before doing so, you need a running ELK-Stack, for me I used a docker-compose(ition) from [docker-elk].
For Mac-Users remember the new [docker-machine] superseeding boot2docker.

Testing your configured ELK-Stack is easy by using [SOAP-UI]´s Load-Test-Feature.

After having set up your ELK-Stack and logs are transferred via logstash into Elasticsearch and you activated SOAP-Message-Logging as shown in the CXF-WebService-Configuration, you for shure want to play aroung with the [Kibana´s Visualisation-Features]. And in the end you also want a [Dashboard] configured to show all your stylish Visualisations. You could end up with something like that:

![Kibana-Dashboard for SOAP-Message Analytics](https://github.com/jonashackt/soap-spring-boot-cxf/blob/master/kibana_SOAP-Analytics_dashboard.png)

If if you can´t wait to start or the tutorials are [tldr;], then import my [kibana_export.json](https://github.com/jonashackt/soap-spring-boot-cxf/blob/master/kibana_export.json) as an example.



### Done´s with Loganalysis with ELK-Stack
* Correlate all Log-Messages (Selfmade + ApacheCXFs SOAP-Messages) within the Scope of one Service-Consumer`s Call in Kibana via logback´s [MDC], placed in a Servlet-Filter
* Log SOAP-Messages to logfile (configurable)
* Log SOAP-Messages only to Elasticsearch-Field, not Console (other Implementation)
* Extract SOAP-Service-Method for Loganalysis
* SOAP-Messages-Only logged and formatted for Analysis
* Added anonymize-logstash-filter for personal data in SOAP-Messages (e.g. for production environments in german companies)



## Rules with DMN

A very common problem of projects that implement SOAP-Services is, that an internal Domain-Model differs from the externally defined XML-Schema defined Model, where the JAXB-Classes are generated from. Often according to that the need for Validation of data, that comes from transformation from the SOAP-Message, arises. There are many approaches to do that, e.g. BeanValidation and others. The problem with these straight and easy-at-first approaches is, that the functional complexity is often higher, than thought in the first place. And usually in validating complex SOAP-Requests for Backend-compatibility it is.
For that purpose there´s no silver bullet out there and one has to choose the right thing for this particual problem. One approach, that I implement, was the use of a neat small but yet powerful Rulesengine - not one of theses huge complex one´s, that [Martin Fowler mentioned](http://martinfowler.com/bliki/RulesEngine.html). It´s a quite young one: [camunda´s DMN Engine](https://github.com/camunda/camunda-engine-dmn) which implements OMG´s [DMN-Standard](http://www.omg.org/spec/DMN/)

In our Usecase we have fields, that have to be checked for internal purposes and rules to apply onto these. So I decided to go with two DMN-Decision-Tables:

![WeatherFields2Check-DMN](https://github.com/jonashackt/soap-spring-boot-cxf/blob/feature/plausibility-check-with-DMN/weatherFields2CheckDMN.png)

![WeatherRules-DMN](https://github.com/jonashackt/soap-spring-boot-cxf/blob/feature/plausibility-check-with-DMN/weatherRulesDMN.png)


## Todo's

* Spring Boot Starter CXF
* Functional plausibility check of request-data with [decision tables]
* Configure Servicename in logback.xml from static fields
* Fault Tolerance with Hystrix (e.g. to avoid problems because of accumulated TimeOuts)



[Spring]:https://spring.io
[Spring Boot]:http://projects.spring.io/spring-boot/
[Spring WS]:http://projects.spring.io/spring-ws/
[Apache CXF]:http://cxf.apache.org/
[JAXB Java-XML-Binding]:http://en.wikipedia.org/wiki/Java_Architecture_for_XML_Binding
[SOAP-UI]:http://www.soapui.org/
[jaxws-maven-plugin]:https://jax-ws-commons.java.net/jaxws-maven-plugin/
[JAX-WS-commons project]:https://jax-ws-commons.java.net/spring/
[Weather-Service]:http://wsf.cdyne.com/WeatherWS/Weather.asmx
[Tomcat]:http://tomcat.apache.org/
[decision tables]:https://en.wikipedia.org/wiki/Decision_table
[Profile-Mechanism]:http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html
[fuck you up]:https://github.com/jonashackt/springbootreadfilejar
[ELK-Stack]:https://www.elastic.co/products
[logstash-logback-encoder]:https://github.com/logstash/logstash-logback-encoder/tree/logstash-logback-encoder-4.5
[docker-elk]:https://github.com/jonashackt/docker-elk
[docker-machine]:https://docs.docker.com/machine/get-started/
[Pdfbox]:https://pdfbox.apache.org/index.html
[base64gular]:https://github.com/jonashackt/base64gular
[MDC]:http://logback.qos.ch/manual/mdc.html
[Heroku]:https://www.heroku.com/home
[blogpost]:https://blog.codecentric.de/en/2015/10/deploying-spring-boot-applications-to-heroku/
[here]:https://soap-spring-boot-cxf.herokuapp.com/soap-api
[Kibana´s Visualisation-Features]:https://www.timroes.de/2015/02/07/kibana-4-tutorial-part-3-visualize/
[Dashboard]:https://www.timroes.de/2015/02/07/kibana-4-tutorial-part-4-dashboard/
[tldr;]:https://en.wiktionary.org/wiki/TLDR