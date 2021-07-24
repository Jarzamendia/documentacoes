# Monitoramento

Existem varias formas de monitorar um cluster do Kubernetes, eu particularmente gosto de usar o ELK Stack.

O Prometheus, juntamente com os utilitarios de exportação de metricas e o Grafana como frontend também é uma otima forma de monitoramento.

## Prometheus no Minikube

Iniciando o Minikube para testes

```
$ minikube start

```

Criando o namespace para o Prometheus

```
$ kubectl create namespace prometheus
```

Adicionando o Prometheus Operator

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install prometheus prometheus-community/kube-prometheus-stack --namespace prometheus

$ helm uninstall prometheus --namespace prometheus
```




Verificando a instalação...
```
$ kubectl get pods -n prometheus
```

Acessando o Grafana

```
$ kubectl port-forward -n prometheus service/prometheus-operated 9090:9090
$ kubectl port-forward -n prometheus service/prometheus-grafana 80:3000
$ kubectl port-forward -n prometheus service/prometheus-kube-prometheus-alertmanager 9093:9093


```


```
$ kubectl port-forward -n prometheus service/prometheus-operated 9090:9090
$ kubectl port-forward -n prometheus service/prometheus-grafana 80:3000
```

```
$ kubectl get secret -n prometheus prometheus-grafana -o yaml
```


admin:prom-operator
