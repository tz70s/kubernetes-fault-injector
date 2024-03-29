# Kubernetes Fault Injection for a single Pod

### Principle

1. Each Kubernetes' pod contains several user's containers attach on a system-default "pause" container with sharing network namespace.That is, *each containers in a pod can easily use localhost to communicate.*
2. Fault Injector masquerade the response from original responsed web server, and using L7 Reverse Proxy make normal traffic pass.
3. Async Web Server with handling route and query string to change fault injection policy

```BASH
#
############################################
#####            ############           ####
#####  FAULT     ############  TESTED   ####
##### INJECtION  ############   HTTP    ####
#####  L7PROXY   ############  SERVER   ####
########  -----  ############  ------   ####
########       container/pause          ####
########                                ####
############################################
```

### Functionality
This work reveal simple HTTP fault injection actions before arriving at the endpoint container.
* Simply change fault injection policy over simple HTTP GET with query string and relative url route 
* 80% for normal traffic and 20% for tested traffic based on L7 Reverse Proxy and route handler

### Kubernetes plugin
* Polling Kubernetes API server and autmatically add fault-injector container in user-defined pod
```BASH
1. Retrieve/GET api server as a standalone microservice
2. GET label selected fault-injector pod
#  Labels:
#    injector: false
3. Replace original pod to new pod including fault-injector container and original containers
4. Automatically integrate original container expose port to reverse proxy backend port
```

### Current Functions
* simpleResponse
```BASH
curl -qa http://inject.default.svc.cluster.local:8282/injector?policy=simpleResponse
```
* timeoutTest/delayResponse
Set timout, default is 10 secs
```BASH
curl -qa http://inject.default.svc.cluster.local:8282/injector?timeout=10
```
* statusResponse
Set status code you like
```bash
curl -qa http://inject.default.svc.cluster.local:8282/injector?status=404
```
* boundedRetries
Set bound of retries, default is 10 retries
```bash
curl -qa http://inject.default.svc.cluster.local:8282/injector?boundedRetries=10
```
