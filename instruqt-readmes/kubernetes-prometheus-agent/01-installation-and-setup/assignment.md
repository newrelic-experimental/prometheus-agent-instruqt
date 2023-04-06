---
slug: installation-and-setup
id: x55h1wjg7wdp
type: challenge
title: 1. Installation and Setup
teaser: Install the Prometheus Agent and other New Relic components into your cluster
notes:
- type: text
  contents: |-
    **Please wait while we build out your Kubernetes cluster.**

    This is a good time to retrieve your New Relic Account ID, License Key, and User API Key.  You'll need them in the first lab challenge.
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
- title: New Relic Platform
  type: website
  url: https://one.newrelic.com/
  new_window: true
difficulty: basic
timelimit: 900
---
Overview
===

###  **Objective:** Get things installed!

---

### **A little bit about your Kubernetes cluster:**

While the wheel was spinning, a single node Kubernetes cluster was created using [Minikube](https://minikube.sigs.k8s.io/docs/). In addition, the NGINX Ingress Controller (a simple source of Prometheus metrics) was added to give you some metrics to work with.

It's a good idea to do a quick check to ensure everything is up and running.

Run the following command:

```
kubectl get pods -A
```

Your cluster should look something like this:

![kubectl](https://p191.p3.n0.cdn.getcloudapp.com/items/rRugng1D/32d6b07d-ea6e-4ec8-b333-4ec0a37fc717.jpg?source=viewer&v=babd92dc9aecec85c0a65873f957d0c8)

If it doesn't, wait a minute and try again.  It shouldn't take long for all components to enter a `Running` state.

Installation instructions
===

Let's install the Prometheus Agent and other New Relic components.  In this step, you'll need:

- New Relic Account ID
- New Relic License Key
- New Relic User API Key (starts with NRAK)

If you don't know where to find these items, click on your avatar in the bottom left and select `API Keys`.

![apikeys](https://p191.p3.n0.cdn.getcloudapp.com/items/X6uKO7bn/af9f789b-a69f-44e6-ac44-fa9d28c1451f.jpg?v=7a6cc2303cc63704ff6a7155becd807d)

All 3 items are available in this UI.

![keys](https://p191.p3.n0.cdn.getcloudapp.com/items/GGu7k5KP/8558a2b0-6a4c-4adc-980a-9cebedcfa09e.jpg?v=82e08fcd64f92c847edffdac85b64561)

Run the `install.sh` script and enter your information into the prompts to install New Relic into your cluster.

```
./install.sh
```

You should see some output similar to this:

![install](https://p191.p3.n0.cdn.getcloudapp.com/items/9ZuKd4j2/226fa774-d184-4934-9919-914b22a9e800.jpg?v=092d5a9cf748fde5af97d4fcef05fa20)

If you get an error or you need to clean up and start over for any reason, run the `cleanup.sh` script.

```
./cleanup.sh
```

If you're curious what was installed, you can run `cat ./install.sh` to see the commands that were used.  (Yes, the bash script could probably use some additional input validation and error checking...)

Validate your install
===

If everything went according to plan, you should be able to see all of the New Relic components installed under the `newrelic` namespace.

You can watch the install progress of the pods with this command:
```
watch kubectl get pods -n newrelic
```

When everything is complete, your results should look something like this:
![](https://lh3.googleusercontent.com/pw/AMWts8CFTPqI9LYtpaANjcq_yY4ldQCWfljaIzfYzDcNkj3IgzNEAd-QygIjSZY8TREaELyptuBYB1HkyseHLp1thWjOWwTMCIiZ8_wcul0eqe8tDSjcx0bFLqMWBOATOk14LE53ab0vavKlYCoZGm2zbHPc=w1502-h394-no?authuser=0)

Use `ctrl + c` to cancel out of the `watch` command.

In addition to the components installed in the cluster, a dashboard was installed into your New Relic account called `Prometheus Agent Instruqt Lab`.   Double check the `Dashboards` in your New Relic account to validate that this was created successfully.  You'll be referring to it throughout the lab.

The dashboard will look like this:

![Dashboard](https://p191.p3.n0.cdn.getcloudapp.com/items/BlubKqdR/fd47200d-df0d-4ae0-ac86-69cf1a362bff.jpg?v=6edcfa2d55f07502a777db64866693b0)

Recap
===

The install script used Helm to install the following components into your cluster:

- New Relic Prometheus Agent
- New Relic Infrastructure for Kubernetes **
- New Relic Logging
- New Relic Kubernetes Events integration **
- New Relic Metadata Injection **

> ** - not used for this lab but part of the core K8s components from New Relic.
---
**NOTE:** The exercises in this lab were built around modifying the Helm values file for the Prometheus Agent chart (`newrelic-prometheus-configurator`) directly and not the `nri-bundle` umbrella chart.  This keeps the lab a little simpler and less error prone.

Users would normally deploy the umbrella `nri-bundle` chart and configure it to include the Prometheus agent (by setting the `newrelic-prometheus-agent.enabled` to `true` in the `values.yaml` file).

An example [values.yaml](https://github.com/newrelic-experimental/prometheus-agent-instruqt/blob/main/prom-agent/values-nri-bundle.yaml) file for the `nri-bundle` can be referenced [here.](https://github.com/newrelic-experimental/prometheus-agent-instruqt/blob/main/prom-agent/values-nri-bundle.yaml)

---
# **You did it!**

Click the "Check" button so we can make sure everything is running properly, then it's off to the next challenge.