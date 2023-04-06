---
slug: metric-opt-in
id: rgs9gdx4oi2u
type: challenge
title: 3. The water's cold!  Ease your way in.
teaser: Selectively collect Fluenbit Prometheus metrics by implementing a custom scrape
  annotation
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
  workdir: /workdir
- title: YAML Editor
  type: code
  hostname: kubernetes-vm
  path: /workdir/prom-agent-instruqt/prom-agent/values-simple.yaml
difficulty: basic
timelimit: 900
---
Overview
===

### **Objective:** Add the correct annotation to the `newrelic-logging` pod so the `default` scrape job is bypassed and the `newrelic` scrape job discovers it.

---

### The water's cold!  Ease your way in.

In this challenge, you'll diverge away from the `default` scrape job and you'll focus on the `newrelic` scrape job instead.

The `newrelic` scrape job is provided as an example so that customers can choose to use a **custom** scrape annotation instead of the Prometheus default of `prometheus.io/scrape: true`.

It also `disables` the global `integrations_filter` for that job so that the agent administrator and developers running workloads in the cluster now have more control over what targets are scraped by the agent and ultimately what metrics are shipped to New Relic.


The "newrelic" scrape job
===


Click on the `YAML Editor` tab in the upper left of the Instruqt lab and open the `values-simple.yaml` file in the editor.  Navigate to lines `71-79` to view the `newrelic` scrape job.  Note the following:

![newrelic_job](https://p191.p3.n0.cdn.getcloudapp.com/items/v1uPy8yk/13543c03-4b1d-43d3-8159-5a3b811c53bb.jpg?v=8b6109bed577c1b053b204f437f495a9)

- The `job_name_prefix` for this scrape job is `newrelic`.
- `integrations_filter.enabled` is set to false.  This disables the global `integrations_filter` feature discussed in the previous challenge.
- You're still targeting `pods` and `endpoints` for discovery.
- Scrape targets **must contain** the `newrelic.io/scrape: true` annotation in order for this job's discovery to be successful.

Why is this scrape job useful?  You may want to be very selective about which metrics sent to New Relic and therefore, an "opt-in" approach might make managing metric volume a little easier.  For example, as a developer, you could add the `newrelic.io/scrape: true` annotation to the YAML manifest for your application pods.  This would ensure that the `newrelic` scrape job will automatically scrape the exposed Prometheus metrics when you deploy your application to the cluster.

The  `newrelic.io/scrape: true` annotation will not exist natively in Kubernetes unless you add it (or another custom annotation).


Working with custom annotations
===

During the installation process, the install script super-secretly deployed New Relic Logging to your cluster which is running New Relic's pre-configured Fluentbit Daemonset.

Did you know [Fluentbit can be configured](https://docs.fluentbit.io/manual/pipeline/outputs/prometheus-exporter) to expose a Prometheus `/metrics` endpoint containing metrics about the performance of the Fluenbit instances?

If you navigate to the `Prometheus Agent Instruqt Lab` dashboard, you'll see that none of the Fluentbit metrics are currently being scraped and sent to your New Relic account.

![fluenbit_metrics](https://p191.p3.n0.cdn.getcloudapp.com/items/yAuJ4mmO/354ae4ee-d3fc-4921-933b-3f8bc6836aab.jpg?v=83213441a2812b2a7dc321263cd1740f)

In the following steps, you'll add the `newrelic.io/scrape` custom annotation to the `newrelic-logging ` pod so that you can bring the `fluentbit` Prometheus metrics into your account.

First, store the `newrelic-logging` pod name in an environment variable by running this command:

```
export LOGGING_POD=$(kubectl get pods -n newrelic -o=jsonpath='{.items[*].metadata.name}{"\n"}' | tr ' ' '\n' | grep logging)
```

Validate that the pod name was stored successfully by printing the environment variable:
```
echo $LOGGING_POD
```

Now query the labels associated with the `newrelic-logging` pod.
```
kubectl get pod $LOGGING_POD -n newrelic --show-labels
```

You should see the following label: `app.kubernetes.io/name=newrelic-logging`.
![logging_pod](https://p191.p3.n0.cdn.getcloudapp.com/items/04u6JE8x/6612084c-b3fd-4b07-ac61-94d3186c7b0c.jpg?v=525801992063264be3e7b1de433f584c)

If you look at line `44` in the YAML Editor, you'll see that `newrelic-logging` is not currently listed as one of the `app_values` within the `integrations_filter`.  This explains why its metrics aren't discovered by the `default` scrape job.

**Remember**: the `default` scrape job requires the `prometheus.io/scrape` annotation **AND** a matching `source_label` and `app_value` within the `integrations_filter`.

![app_values](https://p191.p3.n0.cdn.getcloudapp.com/items/nOuKlRr9/2fff8e7c-d242-42dd-896c-b9e934f65b52.jpg?v=c1d9b29f4ae5dc113efe0052588c5e81)

Apply the `newrelic.io/scrape: true` custom annotation to the `newrelic-logging` pod. This will enable the `newrelic` scrape job to discover the `newrelic-logging` metrics endpoint and collect its metrics.

```
kubectl annotate pod $LOGGING_POD -n newrelic newrelic.io/scrape=true
```

You can validate that the annotation was added successfully with this command:

```
kubectl describe pod $LOGGING_POD -n newrelic | grep -A 3 Annotations
```

You should see `newrelic.io/scrape: true` in the list (among others).

![logging_annotations](https://p191.p3.n0.cdn.getcloudapp.com/items/6qub576B/5cc43865-29a4-4673-acf0-f37504d3d364.jpg?v=8e6eef190a4b68a032b0364c0d81fa64)

It's important to point out that the `prometheus.io/path` and `prometheus.io/port` annotations tell the Prometheus Agent where to find metrics once the endpoint is discovered. For example, by default, the agent looks for metrics at the `/metrics` endpoint but sometimes metrics are exposed at a different path.  This is where the `prometheus.io/path` annotation is helpful.

> It is also possible to customize the `prometheus.io/path` and `prometheus.io/port` annotations, but that is out of scope for this lab.



Validation and Recap
===
After a minute or so, you can navigate to the `Prometheus Agent Instruqt Lab` dashboard and you should see Fluentbit metrics now flowing into your account.  Don't forget to browse the `Promethus Metric List` widget to see the metric names.

![fluentbit_metrics2](https://p191.p3.n0.cdn.getcloudapp.com/items/7KuzxGxB/b346b775-c33f-4644-9f64-fdc0928949c6.jpg?v=71c1fd20a70f5aabb6fe67c0a8aae4c5)

A quick recap of what was just covered:

- The `default` scrape job was not recognizing the Fluentbit metrics because `newrelic-logging` does not exist in the `app_values` for the `integrations_filter`.
- The `newrelic` scrape job was not recognizing the Fluentbit metrics because the `newrelic.io/scrape: true` annotation did not exist on the `newrelic-logging` pod.
- As a user wanting precise control over what metrics are scraped and sent to New Relic, you can use the `newrelic` scrape job and the custom annotation to selectively "opt-in" to certain groups of metrics.
- After applying the `newrelic.io/scrape: true` annotation to the `newrelic-logging` pod, metrics began to flow freely into your New Relic account.

**Congrats!**  You can now proceed to the next challenge.