---
slug: where-are-my-metrics
id: 1qs7bdl1sbbp
type: challenge
title: 2. Where are my NGINX metrics?
teaser: Work with the integrations filter and Kubernetes labels to enable NGINX metrics
  collection
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

### **Objective:** Correctly label the NGINX Ingress pod so it passes the integrations filter validation.

---

### Where are my NGINX metrics?

In Kubernetes, Prometheus can scrape metric endpoints using the [Kubernetes service discovery](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config) mechanism.  This built-in feature enables **automatic** discovery of targets within a Kubernetes cluster.

Prometheus uses [Kubernetes labels and annotations](https://stackoverflow.com/questions/67223202/what-is-the-difference-between-annotations-and-labels-in-in-kubernetes) to identify targets to scrape. Labels and annotations are key-value pairs that can be attached to Kubernetes objects, such as pods, services, and nodes, to provide metadata about the objects.

As mentioned in the previous challenge, an NGINX Ingress Controller was installed when you created your K8s cluster. The Ingress Controller, like many other applications, exposes its metrics using a `/metrics` HTTP endpoint for the Prometheus Agent to scrape.

If you look at your `Prometheus Agent Instruqt Lab` dashboard, you'll notice the `Count of NGINX Metrics Names` widget is currently `0`.  This is because the NGINX Ingress Controller pod does not currently have the Kubernetes labels the Prometheus Agent is configure to look for, therefore, the Prometheus Agent discovery mechanism is unable to "discover" it.

![Missing metrics](https://p191.p3.n0.cdn.getcloudapp.com/items/2NubXPOo/1fad4a31-5c5d-4895-ae70-fdf09c1252dd.jpg?v=1f9e31817f282dbfaa75b8c004047731)

Let's fix that.

Understanding the Integrations Filter
===
Click the `YAML Editor` tab in the upper left of the Instruqt window and select the `values-simple.yaml` file.

![values file](https://p191.p3.n0.cdn.getcloudapp.com/items/NQuWDNq0/29fbee1f-47cc-40f5-a9fa-8ec15e807bb1.jpg?v=aa0a1e9e66aba94ed1a570ce56595ea3)

This is a simplified version of the Prometheus Agent Helm "values" file.  It contains all of the configuration options for instructing the agent where to look for metrics, which metrics to collect, which metrics to keep or drop, and ultimately, where to send them (e.g. New Relic).

Navigate to lines `37-44` in the file and notice the `integrations_filter` section.

![integrations_filter](https://p191.p3.n0.cdn.getcloudapp.com/items/mXuDBYND/05c325b0-5727-44c1-9a15-7c95fa2101b8.jpg?v=c1dbba03cfd6c87e91a875ff0dcd79ce)

- One of the `source_labels` must exist on any discovered pod or endpoint
- `app_values` are the values that the agent looks for in one of the `source_labels` (NOTE: these do not need to be an exact match.  we simply convert these to a regex like `.*nginx.*` for matching purposes).

For example, for NGINX, the Prometheus agent discovery process would match on the following labels:

- `app.kubernetes.io/name: nginx`
- `app.newrelic.io/name: nginx-ingress`
- `k8s-app: nginx-ingress-controller`

The `integrations_filter` is **globally enabled by default** but can be disabled globally or for specific scrape jobs.  This feature acts as an **an additional validation step** prior to remote writing metrics to New Relic and ensures the Prometheus Agent doesn't ship every metric from every endpoint with the `prometheus.io/scrape: true` annotation by default.  Metrics that are not validated by the filter are dropped.

The `source_labels` and `app_values` parameters are used to determine the list of "allowed" integrations and these are also directly aligned with our [Prometheus Quickstarts](https://docs.newrelic.com/docs/infrastructure/prometheus-integrations/integrations-list/integrations-list-intro/).  A Quickstart is a dashboard, examples alerts, and links to related documentation for a specific integration.




Understanding Scrape Jobs
===

Now navigate to lines `60-66` in the file and look for the `default` scrape job.

![scrape_job](https://p191.p3.n0.cdn.getcloudapp.com/items/qGuqd7nR/bce2d096-6cc0-4858-ad0f-cd94d081f746.jpg?v=ea0a956ffed133901d73c968061ffc18)

 Scrape jobs are used to tell the agent what to look for in the cluster and what to do with the metrics once they're discovered.  These can be very simple or very complex. This `default` job is shipped and enabled by default with the Prometheus Agent.

- `job_name_prefix` is the prefix that gets attached to the scrape job name.
- `target_discovery.pod` and `target_discovery.endpoints` tell the agent which type of Kubernetes targets to discover.  The job names for these discovered targets will be `default-pods` and `default-endpoints` respectively.  These job names are attached to Prometheus metrics so it's easy to identify which job discovered the metric.
- `target_discovery.filter.annotations` tells the job which annotations (or labels) to look for in order to "discover" endpoints to scrape.  The `default` scrape job looks for the `prometheus.io/scrape: true` annotation.



Validating annotations and applying labels
===

Run the following command in your terminal to retrieve the NGINX Ingress Controller pod name and store it in an environment variable for later use:
```
export INGRESS_POD=$(kubectl get pods -n nginx-ingress -o=jsonpath='{.items[*].metadata.name}{"\n"}')
```

Quickly validate that the variable has a value:
```
echo $INGRESS_POD
```

The `NGINX Ingress Controller` pod already contains the correct annotations to be discovered by the `default` scrape job.  You can view the existing annotations on the pod by running this command:

```
kubectl describe pod $INGRESS_POD -n nginx-ingress | grep -A 2 Annotations
```

Do you see `prometheus.io/scrape: true` in the list?

Now you can add the appropriate label to the pod(s) so the `default` scrape job will match on the `prometheus.io/scrape: true` annotation **AND** the integrations filter will also match on the `app.newrelic.io/name: nginx` label.

> This is why metrics were not scraped by default and not showing in the dashboard.  The  `app.newrelic.io/name: nginx` label does not exist on the pod.

Run the following command to add the `app.newrelic.io/name: nginx` label to your pod:

```
kubectl label pod $INGRESS_POD -n nginx-ingress app.newrelic.io/name=nginx
```

You can check that the label was added successfully with this command:

```
kubectl get pod -l app.newrelic.io/name=nginx -n nginx-ingress --show-labels
```

 You should see `app.newrelic.io/name: nginx` in the `LABELS` column

![nginx_label](https://p191.p3.n0.cdn.getcloudapp.com/items/E0uXbgQN/fd47ad7c-5f33-4b33-8e28-bc81a44136b2.jpg?v=31801694a95f123f93c2570c61d7bd0f)

**NOTE:** Manually labeling and annotating resources is not "best practice" (it's great for labs though!) as they will be lost if a resource is restarted.  Typically, these are added to the manifest files and maintained as part of your CI pipeline.


Validation and Recap
===

After about a minute, you should see NGINX metrics showing up in the dashboard.  The dashboard will auto-refresh, so just be patient...or you can refresh the browser on your own. You can also browse the `Prometheus Metric List` widget to view a list of all metrics that the Prometheus Agent is sending to your New Relic account.

Do you see any `nginx` metrics in the list?

![nginx_metrics](https://p191.p3.n0.cdn.getcloudapp.com/items/8Luqjd6l/55f1e43a-a1c8-4caa-8eae-f26680840658.jpg?v=755729ec6ffca152ee04fcdbfcfb0add)

A quick recap of what was just covered:

- `scrape jobs` tell the agent what to scrape and how to identify it
- The ` integrations filter` is an extra filter to ensure we only bring in a pre-determined set of metrics by default.  This can be modified and customized by the user by updating the `source_labels` and `app_values` parameters.
- The reason that the NGINX metrics were not immediately showing up is because the NGINX Ingress Controller pod did not have the required `scrape_label` and corresponding `app_value` to satisfy the `integrations_filter`.  Once this was added, the endpoint was discovered and scraped successfully.

**Congrats!**  You should now be more familiar with the integrations filter..  You can now proceed to the next challenge.