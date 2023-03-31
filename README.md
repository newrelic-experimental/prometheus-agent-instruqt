<a href="https://opensource.newrelic.com/oss-category/#new-relic-experimental"><picture><source media="(prefers-color-scheme: dark)" srcset="https://github.com/newrelic/opensource-website/raw/main/src/images/categories/dark/Experimental.png"><source media="(prefers-color-scheme: light)" srcset="https://github.com/newrelic/opensource-website/raw/main/src/images/categories/Experimental.png"><img alt="New Relic Open Source experimental project banner." src="https://github.com/newrelic/opensource-website/raw/main/src/images/categories/Experimental.png"></picture></a>

# Prometheus-Agent-Instruqt

This is supplemental material related to the Prometheus Agent for Kubernetes Instruqt Lab.

## Background

The ability to run Prometheus Server in "Agent Mode" was released with Prometheus Server `v2.32.0`.  You can read more about it [here](https://prometheus.io/blog/2021/11/16/agent/#prometheus-agent-mode).  Subsequently, New Relic released the [Prometheus Configurator](https://github.com/newrelic/newrelic-prometheus-configurator) which is a configuration layer around the Prometheus Agent that automatically configures remote write to New Relic and default scrape jobs.

Learn more about the Prometheus Agent in [our documentation](https://docs.newrelic.com/docs/infrastructure/prometheus-integrations/install-configure-prometheus-agent/setup-prometheus-agent/).

## Installation

This lab is delivered in the Instruqt platform and all instructions are contained in the lab itself.  Access the lab here: https://play.instruqt.com/newrelic/invite/up9jflwp3clf.

## Getting Started

* Create an Instruqt account
* Bring the following: New Relic Account ID, New Relic License Key, New Relic User API Key


### Lab Sections

1. Installation and Setup
    * Install the Prometheus Agent and other New Relic components into your cluster
2. Where are my NGINX metrics?
    * Work with the integrations filter and Kubernetes labels to enable NGINX metrics collection
3. The water's cold! Ease your way in
    * Selectively collect Fluenbit Prometheus metrics by implementing a custom scrape annotation
4. Get those sweet, sweet metrics from a static target
    * Learn how to configure the Prometheus Agent to scrape a static target running on the Instruqt VM
5. Metrics and Metric Labels: How to drop them and pretend it was an accident (wink wink)
    * Learn how to drop metrics and metric labels if cardinality or ingest volume is a concern
6. Who doesn't love a good dashboard?
    * Deploy the CoreDNS Quickstart dashboard to your New Relic account

## Contributing

We encourage your contributions to improve **Prometheus-Agent-Instruqt**! Keep in mind when you submit your pull request, you'll need to sign the CLA via the click-through using CLA-Assistant. You only have to sign the CLA one time per project.
If you have any questions, or to execute our corporate CLA, required if your contribution is on behalf of a company,  please drop us an email at opensource@newrelic.com.

**A note about vulnerabilities**

As noted in our [security policy](../../security/policy), New Relic is committed to the privacy and security of our customers and their data. We believe that providing coordinated disclosure by security researchers and engaging with the security community are important means to achieve our security goals.

If you believe you have found a security vulnerability in this project or any of New Relic's products or websites, we welcome and greatly appreciate you reporting it to New Relic through [HackerOne](https://hackerone.com/newrelic).

## License

**Prometheus-Agent-Instruqt** is licensed under the [Apache 2.0](http://apache.org/licenses/LICENSE-2.0.txt) License.