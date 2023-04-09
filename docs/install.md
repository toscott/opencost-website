---
sidebar_position: 2
---
# OpenCost Setup

OpenCost requires Prometheus for scraping metrics and data storage. Follow the steps below to install OpenCost.

## Quick Start Installation

These commands will get you started immediately with OpenCost.

### Prerequisite: Prometheus
There are multiple support ways to install Promethues. A simplified example is provided below

```sh
helm install my-prometheus --repo https://prometheus-community.github.io/helm-charts prometheus \
  --namespace prometheus --create-namespace \
  --set pushgateway.enabled=false \
  --set alertmanager.enabled=false
```

### Install OpenCost

```sh
helm install opencost --repo https://opencost.github.io/opencost-helm-chart opencost \
  --namespace opencost --create-namespace \
  --set opencost.prometheus.internal.serviceName=my-prometheus \
  --set opencost.prometheus.internal.namespaceName=prometheus
```

That is all that is required for most installations. You can proceed to [testing](#testing) for verifying your installation.

For a more detailed setup tutorial, continue to the next section.

#### Using a prometheus instance outside the cluster
1. Enable external prometheus during install.
```sh
helm install opencost --repo https://opencost.github.io/opencost-helm-chart opencost \
  --namespace opencost --create-namespace \
  --set opencost.prometheus.external.enabled="true" \
  --set opencost.prometheus.external.url="https://prometheus.example.com/prometheus"
```
2. Add the [scrapeConfig](https://raw.githubusercontent.com/opencost/opencost/develop/kubernetes/prometheus/extraScrapeConfigs.yaml) to it.

## Testing

Once your OpenCost has been installed, wait for the pod to be ready and port forward with:

```sh
kubectl port-forward --namespace opencost service/opencost 9003 9090
```

To verify that the UI and server are running, you may access the OpenCost UI at [http://localhost:9090](http://localhost:9090).

To verify that the server is running, access [http://localhost:9003/allocation/compute?window=60m](http://localhost:9003/allocation/compute?window=60m)

You can see more [API Examples](./api.md) or use [kubectl cost](./kubectl-cost.md):

```sh
kubectl cost --service-port 9003 --service-name opencost --kubecost-namespace opencost --allocation-path /allocation/compute  \
    namespace \
    --window 5m \
    --show-efficiency=true
```

Output:

```
+---------+---------------+--------------------+-----------------+
| CLUSTER | NAMESPACE     | MONTHLY RATE (ALL) | COST EFFICIENCY |
+---------+---------------+--------------------+-----------------+
|         | opencost      |          18.295200 |        0.231010 |
|         | prometheus    |          17.992800 |        0.000000 |
|         | kube-system   |          11.383200 |        0.033410 |
+---------+---------------+--------------------+-----------------+
| SUMMED  |               |          47.671200 |                 |
+---------+---------------+--------------------+-----------------+
```

## Updating OpenCost

```sh
helm upgrade opencost --repo https://opencost.github.io/opencost-helm-chart opencost --namespace opencost
```

Check logs to verify the version of your OpenCost:

```sh
$  kubectl logs -n opencost deployment/opencost | head
2022-09-02T18:20:34.327989163Z ??? Log level set to info
2022-09-02T18:20:34.328206357Z INF Starting cost-model (git commit "x.xx.x")
```

### Sidegrading OpenCost
If you wish to modify OpenCost to a specific version, you can supply the opencost version as the image tag:
```sh
helm install opencost --repo https://opencost.github.io/opencost-helm-chart opencost \
  --namespace opencost --create-namespace \
  --set opencost.prometheus.internal.serviceName=my-prometheus \
  --set opencost.prometheus.internal.namespaceName=prometheus \
  --set opencost.exporter.image.tag="1.102.0"
```

Check logs to verify the version of your OpenCost:

```sh
$  kubectl logs -n opencost deployment/opencost | head
2022-09-02T18:20:34.327989163Z ??? Log level set to info
2022-09-02T18:20:34.328206357Z INF Starting cost-model (git commit "x.xx.x")
```

## Deleting OpenCost
To delete OpenCost, enter the following command:

```sh
helm --namespace opencost delete opencost
```

## Troubleshooting

If you get an error like this, check your Prometheus target is correct in the OpenCost deployment.

```bash
Error: failed to query allocation API: failed to port forward query: received non-200 status code 500 and data: {"code":500,"status":"","data":null,"message":"Error: error computing allocation for ...
```

Negative values for idle: ensure you added the scrape target (above) for OpenCost.

## Enabling Debugging

With the [v1.100 release](https://github.com/opencost/opencost/releases/tag/v1.100.0) you can temporarily set the log level of the OpenCost container without restarting the Pod. You can send a POST request to /logs/level with one of the valid log levels. This does not persist between Pod restarts, Helm deployments, etc. Here's an example:
```sh
curl -X POST \
    'http://localhost:9003/logs/level' \
    -d '{"level": "debug"}'
```
A GET request can be sent to the same endpoint to retrieve the current log level.

---

## Help

Please let us know if you run into any issues, we are here to help!

Contact us via email (<opencost@kubecost.com>) or join us on [CNCF Slack](https://slack.cncf.io/) in the [#opencost](https://cloud-native.slack.com/archives/C03D56FPD4G) channel if you have questions!
