# Neotys Selenium server

A version of Selenium Grid which includes Neotys-specific logic to re-use functional tests
 for conversion to protocol-based load test scripts and for synthetic sampling of end-user
 metrics during a load test.

## Why This Project Exists

Measuring the impact on key user-experience metrics of load on a web application is important.
 Many organizations have functional test scripts written in various languages that can easily
 be adapted to run on a standard Selenium Grid, often already coming with a harness that supports
 this mode.

Running them on the Neotys Selenium Grid allows these key metrics to flow into NeoLoad during
 a load test. Additionally, they can also be run in Design mode to automate the traffic recording
 process to produce highly scalable protocol-based test scripts (NeoLoad User Paths) so that the
 'pressure' put on the system is realistic to the same workflow(s) being measured for user-experience.

## Prerequisites

1. Java 8 or higher
2. NeoLoad Desktop client (https://neotys.com/)
3. An activated license that includes module 'Integration and Advanced Usage'

## Usage

To run this grid with included NeoLoad logic, you need to:

A) Start the jar in hub mode
```
java -jar neotys-selenium-server-[version]-all.jar -role hub
```
B) Start one or more nodes using the same jar in a separate process (terminal)
```
java -jar neotys-selenium-server-[version]-all.jar -role node -hub http://localhost:4444 -proxy com.neotys.selenium.server.NeoLoadRemoteProxy
```
C) Run WebDriver scripts against this hub with additional capabilities specified.

### Running with Docker Compose

You can also use Docker Compose to run an example of this grid. However, this specifically requires a few key configuration points:

1. Run your script using --headless mode (see example test suite for Mocha and Protractor setup) otherwise you'll see an error when trying to run your scripts ['Cannot assign requested address (99)'](https://github.com/RobCherry/docker-chromedriver/issues/15#issuecomment-426367228). Headless is required because this Docker-based grid uses a container-based version of the official Chrome Selenium node image which does not contain a Window Manager on purpose.

2. Override the neoload:host IP address to be the NIC IPv4 address of your host, so that the containerized hub can interact with NeoLoad APIs on the host. In the below examples, the test suites have been written to accept command line arguments 'neoloadHost' to marshal the provided value into a custom capabilities entry for this purpose.

3. Docker dependencies assume that you've built the jar inside this repo, but you can simply download a released/compiled version of the jar from [this repository's release assets](https://github.com/paulsbruce/neotys-selenium-server/releases/).

## Examples

You can use/adapt NeoLoad [examples in this repo](https://github.com/paulsbruce/NeoLoadSeleniumExamples.git), or create your own with WebDriver and layer in a few minimum-viable additional capabilities that help this version of the Grid know which mode (EndUserExperience or Design) you wish to run the test under.

Specifically, [Mocha](https://github.com/paulsbruce/NeoLoadSeleniumExamples/tree/master/custom-resources/selenium/tests/mocha) and [Protractor](https://github.com/paulsbruce/NeoLoadSeleniumExamples/tree/master/custom-resources/selenium/tests/protractor) based examples are provided in the above repo.

## Additional Capabilities in WebDriver

The standard way to specify implementation-specific details in the WebDriver protocol is to use custom capabilities, such as:
```
capabilities = {
  'neoload:mode': 'EndUserExperience',
  'neoload:host': 'my.controller.domain.com',
  'neoload:port': '7400'
}
```

For information that is derived *during* test steps, such as script name and transaction names, the NeoLoad Selenium Grid expects that cookies and/or javascript executes are used to courier context from the script client to the hub. Unfortunately, there is no current standard adopted yet by W3C WebDriver that accomplishes this use case. However, use of cookies and script executes are not passed to the actual browser/client. Also, this functionality is supported by all WebDriver frameworks and client libraries, so is very portable between test suites and harnesses.

## Corporate Proxies

Sadly, they have to exist. If a proxy is required between your hub and the NeoLoad controller, add the following capabilities (like the above):
```
capabilities = {
  ... some other NeoLoad capabilities ...

  'neoload:proxyHost': 'your_proxy_hostname_or_ip_address',
  'neoload:proxyPort': your_proxy_port_number,
  'neoload:nonProxyHosts': 'the_hostname_of_your_selenium_hub|the_ip_address_of_your_selenium_hub'  
}
```
The value of 'nonProxyHosts' is important unless you don't mind ALL selenium hub traffic INCLUDING hub-to-nodes request to go through this proxy, but this is usually not the case. Usually, you want direct interactions between hub and nodes.

There are also keys 'neoload:proxyUser' and 'neoload:proxyPassword' if your proxy requires authentication.

Finally, you can try to use 'neoload:useSystemProxies' and set to true if you feel brave enough to rely on the operating system to dictate what proxy information to use (but I personally like to be explicit).


## Obtaining Help

DISCLAIMER: This information and all assets are provided with no official support by Neotys, but rather, is a prototype by the author(s) and constitutes no legal obligation or statement of liability.

If you want to play around with it, feel free, but it's at your own risk. If your scripts cause data loss or operational issues in corporate environments, that's on you.

If you'd like to discuss this topic, please DM [@paulsbruce](https://twitter.com/paulsbruce)

## Future Work

TOPIC: hub-specific spin-up arguments for mode/host/port so that RemoteWebDriver scripts don't require any changes
TOPIC: integration into a Selenoid farm; add jar to classpath of hub and to -proxy option of node Docker image
TOPIC: add node capabilities matcher so that a variety of nodes can be run on the same grid
TOPIC: expose key metrics using [OpenTelemetry](https://opentelemetry.io) protocol; ingestion by Neotys OTel collector
TOPIC: deeper injection/extraction of Chrome and Firefox specific Profiler data (function-level performance)
TOPIC: more reference examples for Java, C#, Python, Ruby, etc.
