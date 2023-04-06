---
slug: drop-metrics-and-labels
id: udjy9opw2odc
type: challenge
title: '5. Metrics and Metric Labels: How to Drop Them and Pretend It Was an Accident
  (Wink Wink)'
teaser: Learn how to drop metrics and metric labels if cardinality or ingest volume
  is a concern
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
- title: YAML Editor
  type: code
  hostname: kubernetes-vm
  path: /workdir/prom-agent-instruqt/prom-agent/values-simple.yaml
difficulty: basic
timelimit: 600
---
Overview
===

### **Objective:** Enable the metric relable configs to drop metrics and labels before shipping to New Relic.

---

### Metrics and Labels: How to Drop Them and Pretend It Was an Accident (Wink Wink)

Ok!  You've made it this far.  Metrics are flowing.  Things are looking good.  Now let's start making changes to the metrics being sent to your account.  Wait, you just got this working, why would you want to make changes?!

Often times, you may not want to remote write every possible metric to New Relic.  You may want to drop a small subset of metrics as they're not important to your overall observability strategy.  You might also need to reduce the overall cardinality of one or more metrics by dropping a specific label associated with a metric.

Prometheus has [the concept of metric relabeling](https://grafana.com/blog/2022/03/21/how-relabeling-in-prometheus-works/) which is a very powerful way of manipulating metrics before they're stored or shipped off to a remote write endpoint, like New Relic.

In this challenge, you'll be implementing some basic metric relabeling examples.  This can be a very deep topic so please be sure to review the link above as well as any supplemental links with this lab.


Uncomment metric_relabel configs
===

Click on the YAML Editor tab in the upper left of the Instruqt lab and open the `values-simple.yaml` file in the editor. Uncomment lines `91 - 99` to enable the `extra_metric_relabel_config` section for the `instruqt_node_exporter` job.

---

**HINT!** - you can highlight all lines that you want to uncomment and use `command` + `/` on your keyboard to uncomment everything at once.

---

Your file should look like this.  **Double check your indentation!**

![relabel_configs](https://p191.p3.n0.cdn.getcloudapp.com/items/E0uXbP2p/d2f68eae-a67b-426c-983f-37ea74303013.jpg?v=8ea7b103c87aa2598b87faf2efff51ce)

Click the disk icon in the YAML Editor to save your file.

![disk_icon](https://p191.p3.n0.cdn.getcloudapp.com/items/E0uXb7zl/6af5ee1a-c88c-40b1-a2c5-8a3733b8ff16.jpg?v=a1f61af0d81ec1aa5832feda660b453f)

Run the `helm upgrade` command to update the Prometheus Agent config and redeploy the agent:

```
helm upgrade --install prometheus-agent newrelic-prometheus/newrelic-prometheus-agent -f ../prom-agent/values-simple.yaml -n newrelic
```

While the update is taking place, read on to learn about the `metric_relabel` rules that you just enabled.

Dropping Metrics
===

In this metric relabel config, you're:

- evaluating **all metric names** indicated by `__name__` in the `source_labels` list.  `__name__` is a Prometheus label that contains the full metric name.
- the `regex` field is then used to identify which metrics will be "in scope" for this specific relabel config.  In the example, you're targeting all metrics with the `node_socket` and `node_timex` prefixes.
- the `action` is used to tell the agent what to do with metric names that match the `regex`.  In the example, you're dropping all metrics that match.

![drop_metrics](https://p191.p3.n0.cdn.getcloudapp.com/items/9ZuKRJRw/aac92e43-a866-4f6c-bb5f-8e054ab69ad0.jpg?v=7595c591c0eb0200e376a6802ab8d89b)


Dropping Metric Labels
===

In this metric relabel config, you're:

- evaluating **all metric names** indicated by `__name__` in the `source_labels` list **AND** the `device` label.  `__name__` is a Prometheus label that contains the full metric name.
- the `regex` field is then used to identify which metrics will be "in scope" for this specific relabel config.  In the example, you're targeting all `node_network` metrics.
- A second `regex` is provided to match on the `ens4` and `cni0` devices.
- The two `regexes` for the metric name and device are separated with a semicolon `;`.  This is the default separator but you can specify your own with the `separator:` field (not used in this lab).
- the `action` is used to tell the agent what to do with metric names that match the `regex`.  In the example, the action is `keep` which means for all `node_network` metrics, you'll drop all `devices` except `ens4` and `cni0`.

![drop_metrics_and_labels](https://p191.p3.n0.cdn.getcloudapp.com/items/rRugWAjK/d8a0e30f-07a0-4ce0-b387-5a21161b8463.jpg?v=bf1f6bd09612419edfd889a030119d2c)

Recap
===

You've only scratched the surface when it comes to metric relabeling but as you can see, it can be a very powerful tool to manage and manipulate metric data within the agent.  After the changes you implemented above, you should see a sharp drop in the count of metrics ingested in your dashboard.

![metric_drop](https://p191.p3.n0.cdn.getcloudapp.com/items/Apuvjyyv/264136df-a29b-4592-839f-cf0dc141aa85.jpg?v=f45f0854fc7fb380d7e86a5a71633201)

You'll also see a drop in overall cardinality for the `node_network` metrics after we dropped **all but two** `device` labels ( `ens4` and `cni0`).  The additional devices (14 in total) were contributing to high cardinality.  And finally, you'll notice that the `node_sockstat` and `node_timex` metrics are no longer being ingested.

![no_more_metrics](https://p191.p3.n0.cdn.getcloudapp.com/items/YEue1vvE/c76740b2-c752-4fc9-ae47-1129090f54a7.jpg?v=b112c12e4940de86d7c97fcabbdf1a95)

These are common examples of how you can **send the right data** to your New Relic account without shipping data that you don't care about.

Click the **Next** button to continue!