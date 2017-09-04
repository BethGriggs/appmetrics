# appmetrics-prometheus

<!-- [![Build Status](https://travis-ci.org/RuntimeTools/appmetrics-dash.svg?branch=master)](https://travis-ci.org/RuntimeTools/appmetrics-dash)
[![codebeat badge](https://codebeat.co/badges/52b7334d-70b0-4659-9acb-b080d6413906)](https://codebeat.co/projects/github-com-runtimetools-appmetrics-dash-master)
[![codecov.io](https://codecov.io/github/RuntimeTools/appmetrics-dash/coverage.svg?branch=master)](https://codecov.io/github/RuntimeTools/appmetrics-dash?branch=master)
![Apache 2](https://img.shields.io/badge/license-Apache2-blue.svg?style=flat)
[![Homepage](https://img.shields.io/badge/homepage-Node%20Application%20Metrics-blue.svg)](https://developer.ibm.com/node/monitoring-post-mortem/application-metrics-node-js/) -->

appmetrics-prometheus provides a /metrics endpoint which is necessary for [Prometheus monitoring](https://prometheus.io/).

The data available on the /metrics endpoint is as follows:
* CPU
* Memory

appmetrics-prometheus uses [Node Application Metrics][1] to monitor the application.

## Configuring Prometheus

### Local Installation

Follow the instructions on the [Prometheus getting started](https://prometheus.io/docs/introduction/getting_started/) page.

Or follow the simple example below.

Install Prometheus using:

```
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```
Next you need to create a configuration file for Prometheus to use.
In this you can set the scrape interval and set the targets that Prometheus will get the metrics of.
Create a file named prometheus.yml

```
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  external_labels:
    monitor: 'codelab-monitor'
    
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'YOUR JOB NAME'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['YOUR URL', 'YOUR OTHER URL']
```

Set the targets field to your url (or urls adding a comma between each one).

Start Prometheus by using the command

```
./prometheus -config.file=prometheus.yml
```
Prometheus can be found at `localhost:9090`

### Kubernetes

To use Prometheus with Kubernetes you can install it using [Helm](https://github.com/kubernetes/helm).

[Prometheus Chart](https://github.com/kubernetes/charts/tree/master/stable/prometheus)

`$ helm install stable/prometheus`

## Installation

```console
npm install appmetrics-prometheus
```

## Performance overhead

Our testing has shown that the performance overhead in terms of processing is minimal, adding less than 0.5 % to the CPU usage of your application.

We gathered this information by monitoring the sample application [Acme Air][3]. We used MongoDB as our datastore and used JMeter to drive load though the program.  We have performed this testing with Node.js version 6.10.3

## prometheus = require('appmetrics-prometheus').monitor()

This will launch the prometheus endpoint and start monitoring your application.
The prometheus metrics page is located at /metrics.

Simple example using the express framework

```js
// This application uses express as its web server
// for more info, see: http://expressjs.com
var express = require('express');

var prometheus = require('appmetrics-prometheus').attach();

// cfenv provides access to your Cloud Foundry environment
// for more info, see: https://www.npmjs.com/package/cfenv
var cfenv = require('cfenv');

// create a new express server
var app = express();

// serve the files out of ./public as our main files
app.use(express.static(__dirname + '/public'));

// get the app environment from Cloud Foundry
var appEnv = cfenv.getAppEnv();

// start server on the specified port and binding host
var server = app.listen(appEnv.port, '0.0.0.0', function() {
	// print a message when the server starts listening
  console.log("server starting on " + appEnv.url);
});
```

## prometheus.attach(options)

* options.appmetrics {Object} An instance of `require('appmetrics')` can be
  injected if the application wants to use appmetrics, since it is a singleton
  module and only one can be present in an application. Optional, defaults to
  the appmetrics dependency of this module.

Auto-attach to all `http` servers created after this call, calling `prometheus.monitor(options)` for every server.

Simple example using attach
```js
require('appmetrics-prometheus').attach();

var http = require('http');

const port = 3000;

const requestHandler = (request, response) => {  
  response.end('Hello')
}

const server = http.createServer(requestHandler);

server.listen(port, (err) => {  
  if (err) {
    return console.log('An error occurred', err)
  }
  console.log(`Server is listening on ${port}`)
});
```

## Contributing

We welcome contributions. Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details about the contributor licence agreement and other information. If you want to do anything more involved than a bug fix or a minor enhancement then we would recommend discussing it in an issue first before doing the work to make sure that it's likely to be accepted. We're also keen to improve test coverage and may not accept new code unless there are accompanying tests.

### License
The Node Application Metrics Prometheus is licensed using an Apache v2.0 License.


[1]:https://developer.ibm.com/open/node-application-metrics/
[2]:https://www.npmjs.com/package/node-report/
[3]:https://github.com/acmeair/acmeair-nodejs/
