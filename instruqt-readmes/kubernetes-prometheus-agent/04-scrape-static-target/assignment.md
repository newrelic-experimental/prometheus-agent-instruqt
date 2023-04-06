---
slug: scrape-static-target
id: vvxaq8rzylja
type: challenge
title: 4. Get those sweet, sweet metrics from a static target
teaser: Learn how to configure the Prometheus Agent to scrape a static target running
  on the Instruqt VM
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
- title: YAML Editor
  type: code
  hostname: kubernetes-vm
  path: /workdir/prom-agent-instruqt/prom-agent/values-simple.yaml
difficulty: basic
timelimit: 900
---
Overview
===

### **Objective:** Learn how to configure the Prometheus Agent to scrape a static target running on the Instruqt VM.

---
### Get those sweet, sweet metrics from a static target

The [Prometheus Node Exporter](https://github.com/prometheus/node_exporter) has now been installed directly on to the Instruqt VM (outside of the K8s cluster) and we will use this as a `static target` for this challenge.

In previous challenges, you used Kubernetes labels and annotations to dynamically discover Prometheus endpoints in the cluster.  Sometimes, endpoints will be at a static IP address or DNS name located internally or externally to the cluster.  Examples of this might be an external database server or a cloud-based solution like [Confluent Cloud](https://docs.confluent.io/cloud/current/monitoring/metrics-api.html#ccloud-metrics).

In this challenge, you'll configure the Prometheus Agent to connect to the Prometheus Node Exporter `/metrics` endpoint using the public DNS name for your Instruqt VM.  The Node Exporter endpoint also uses `basic authentication` so you'll be including a username and password as part of the configuration.

One of the benefits of the Prometheus Agent is that there are multiple options for authentication including basic auth, TLS, and Oauth2.


Testing the static target
===


In order to scrape the Node Exporter endpoint exposed on the Instruqt VM, you'll need to identify the public DNS name for your host.  Run the following command to output your public DNS name.

```
echo $HOSTNAME.$_SANDBOX_ID.instruqt.io
```

View Node Exporter metrics using `curl`:

```
curl -s http://$HOSTNAME.$_SANDBOX_ID.instruqt.io:9100/metrics
```

Do you see an `Unauthorized` message?  Don't worry, that's expected.

The Node Exporter endpoint has been configured with basic authentication so unless you provide a username and password, you won't be able to see metrics.  Try this `curl` command instead:

```
curl -s -u prometheus:skofy24prometheus http://$HOSTNAME.$_SANDBOX_ID.instruqt.io:9100/metrics | grep node
```

Much better!  You should see some output that looks like this:
![prometheus_output](https://p191.p3.n0.cdn.getcloudapp.com/items/BlubZ0JG/fd21087c-e3ae-42c7-b49e-6b6e2a5e306d.jpg?v=bd25256a8999c8b3d7093742104eb66a)


Configure the scrape job
===


Click over to the YAML Editor and uncomment lines `84 - 89` in the `values-simple.yaml` file.

- Replace `INSTRUQT_HOSTNAME` with your public DNS name (keep port `9100` in there!).

	If you need the command to find your DNS name again, here it is:

	```
	echo $HOSTNAME.$_SANDBOX_ID.instruqt.io
	```

- Replace `PASSWORD` with `skofy24prometheus`

---

**HINT!** - you can highlight all lines that you want to uncomment and use `command` + `/` on your keyboard to uncomment everything at once.

---


The configuration should look similar to this.  Be careful with indentation!  Double check that everything is aligned properly.  The password should be `skofy24prometheus`.

![static_target](https://lh3.googleusercontent.com/pw/AMWts8AhhSnZ64nY_sfQMu0JlmcVhzKqJYkeZvKKg5SX3atg7-00sP_82lnA16ok2z58YLXC4py7UMtTyeTTJp4hQqWaYzZkJBqW_dOM7E5ceutHaebzqMmosy0ZFD8DbOpwPzh2lCXWcfrf5RlZ7CAStzJB=w982-h376-no?authuser=0)

**VERY IMPORTANT** - click the disk icon to save the file. ![disk icon](https://p191.p3.n0.cdn.getcloudapp.com/items/E0uXb7zl/6af5ee1a-c88c-40b1-a2c5-8a3733b8ff16.jpg?v=a1f61af0d81ec1aa5832feda660b453f)

Now run the following `helm upgrade` command to update your running Prometheus Agent with the new configuration.
```
helm upgrade --install prometheus-agent newrelic-prometheus/newrelic-prometheus-agent -f ../prom-agent/values-simple.yaml -n newrelic
```

You should see this message once the upgrade has been initiated:

![helm upgrade](https://p191.p3.n0.cdn.getcloudapp.com/items/X6uKOr09/52a0a7a2-ffa6-44e6-aa66-d35aafc1fe69.jpg?v=556aabfb82cab68d1a0838109e78479f)


Validation and Recap
===

After the `helm upgrade`, navigate to your `Prometheus Instruqt Lab` dashboard and you should see the total metric count skyrocket as the `Node Exporter` metrics are collected.

![node_exporter_metrics](https://p191.p3.n0.cdn.getcloudapp.com/items/2Nubrpn2/2d4363bc-3af9-4816-976b-7fb00ad9796c.jpg?v=7a9d9e43345fca607b6ecd9b5d05cf6b)


Congrats!  You've just successfully configured a static target for the Prometheus Agent.

Here's what you just covered:

- `static targets` are endpoints that will remain static, meaning, they don't need to be dynamically discovered and their IP or DNS name is fixed.  Examples can include database servers or cloud-based solutions like Confluent Cloud.
- You can configure more than one target per scrape job and multiple scrape jobs can be configured if needed.

You may now proceed to the next challenge!

Click **Next** to continue.