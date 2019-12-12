# kube-prometheus

```
git clone https://github.com/carlosedp/cluster-monitoring.git
```

```
kubectl create -f cluster-monitoring/manifest/

kubectl create -f cluster-monitoring/manifests/00namespace-namespace.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-0alertmanagerCustomResourceDefinition.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-0podmonitorCustomResourceDefinition.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-0prometheusCustomResourceDefinition.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-0prometheusruleCustomResourceDefinition.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-0servicemonitorCustomResourceDefinition.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-clusterRole.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-clusterRoleBinding.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-deployment.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-service.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-serviceAccount.yaml
kubectl create -f cluster-monitoring/manifests/0prometheus-operator-serviceMonitor.yaml

kubectl create -f cluster-monitoring/manifests/alertmanager-alertmanager.yaml
kubectl create -f cluster-monitoring/manifests/alertmanager-secret.yaml
kubectl create -f cluster-monitoring/manifests/alertmanager-service.yaml
kubectl create -f cluster-monitoring/manifests/alertmanager-serviceAccount.yaml
kubectl create -f cluster-monitoring/manifests/alertmanager-serviceMonitor.yaml

kubectl create -f cluster-monitoring/manifests/grafana-config.yaml
kubectl create -f cluster-monitoring/manifests/grafana-dashboardDatasources.yaml
kubectl create -f cluster-monitoring/manifests/grafana-dashboardDefinitions.yaml
kubectl create -f cluster-monitoring/manifests/grafana-dashboardSources.yaml
kubectl create -f cluster-monitoring/manifests/grafana-deployment.yaml
kubectl create -f cluster-monitoring/manifests/grafana-service.yaml
kubectl create -f cluster-monitoring/manifests/grafana-serviceAccount.yaml
kubectl create -f cluster-monitoring/manifests/grafana-serviceMonitor.yaml

kubectl create -f cluster-monitoring/manifests/kube-state-metrics-clusterRole.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-clusterRoleBinding.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-deployment.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-role.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-roleBinding.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-service.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-serviceAccount.yaml
kubectl create -f cluster-monitoring/manifests/kube-state-metrics-serviceMonitor.yaml

kubectl create -f cluster-monitoring/manifests/node-exporter-clusterRole.yaml
kubectl create -f cluster-monitoring/manifests/node-exporter-clusterRoleBinding.yaml
kubectl create -f cluster-monitoring/manifests/node-exporter-daemonset.yaml
kubectl create -f cluster-monitoring/manifests/node-exporter-service.yaml
kubectl create -f cluster-monitoring/manifests/node-exporter-serviceAccount.yaml
kubectl create -f cluster-monitoring/manifests/node-exporter-serviceMonitor.yaml

kubectl create -f cluster-monitoring/manifests/prometheus-adapter-apiService.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-clusterRole.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-clusterRoleAggregatedMetricsReader.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-clusterRoleBinding.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-clusterRoleBindingDelegator.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-clusterRoleServerResources.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-configMap.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-deployment.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-roleBindingAuthReader.yaml

kubectl create -f cluster-monitoring/manifests/prometheus-adapter-service.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-adapter-serviceAccount.yaml

kubectl create -f cluster-monitoring/manifests/prometheus-clusterRole.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-clusterRoleBinding.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-kubeControllerManagerPrometheusDiscoveryService.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-kubeDnsPrometheusDiscoveryService.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-kubeSchedulerPrometheusDiscoveryService.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-prometheus.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-roleBindingConfig.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-roleBindingSpecificNamespaces.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-roleConfig.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-roleSpecificNamespaces.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-rules.yaml

kubectl create -f cluster-monitoring/manifests/prometheus-service.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceAccount.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceMonitor.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceMonitorApiserver.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceMonitorCoreDNS.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceMonitorKubeControllerManager.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceMonitorKubeScheduler.yaml
kubectl create -f cluster-monitoring/manifests/prometheus-serviceMonitorKubelet.yaml

kubectl create -f cluster-monitoring/manifests/smtp-server-deployment.yaml
kubectl create -f cluster-monitoring/manifests/smtp-server-service.yaml
```



