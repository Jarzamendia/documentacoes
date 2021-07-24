# Monitoramento de Kubernetes usando Prometheus

O Kubernetes tornou extremamente fácil executar grandes cargas de trabalho em ambientes clusterizados de alta disponibilidade. Ao mesmo tempo em que aumentamos o número de aplicações e servidores em nosso ambiente, também aumentamos a complexidade do gerenciamento de nossos ativos.
 
Para ajudar no monitoramento de tanto nossas cargas de trabalho, quanto do próprio ambiente em si, diversas aplicações foram criadas. Algumas delas, como o Dynatrace ou o Datalog são pagas, cobrando pela hospedagem de nossos dados e pelo suporte empresarial oferecido.

Outras, como o Prometheus ou o Elasticsearch são de código aberto, totalmente gratuitas. Mesmo nessas aplicações, também é possível adquirir o suporte empresarial de seus criadores ou usar seus ambientes gerenciados e pagar pelos seus serviços.
 
Por outro lado, também é possível assumir totalmente a gestão e hospedagem dos dados monitorados em nosso ambiente. Para isto podemos usar as opções de código aberto e com isso mitigar os valores pagos a terceiros.
 
## O que é o Prometheus

Uma das plataformas mais usadas pela comunidade é o Prometheus, neste documento iremos tratar de sua instalação em um ambiente de testes do Kubernetes usando o Minikube. O Prometheus é uma plataforma de monitoramento extremamente completa e customizável, por muitos é considerado o sistema de monitoramento padrão da nuvem.

Para ser mais exato, o Prometheus é uma plataforma de código aberto que coleta métricas e as guarda em forma de dados de série temporal. Estes dados podem então ser pesquisados e organizados, de forma a criar gráficos e dashboards conforme nossas necessidades. As aplicações e ambientes devem expor esses dados em endpoints HTTP que serão acessados pelo Prometheus.

Quando tratamos de projetos open source, o Prometheus brilha. O protocolo [OpenMetrics](https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md), criado como uma forma de padronizar o formato de exportação de métricas para o Prometheus é muito difundido na comunidade. Mesmo em aplicações que a princípio não tem suporte a exportar métricas no formato do Prometheus, facilmente conseguimos encontrar extensões da comunidade que podem ser incluídas na aplicação, adicionando então esta funcionalidade.

Junto a isto, criar e editar gráficos e dashboards no Grafana (O visualizador de métricas natural do Prometheus) é extremamente fácil e intuitivo. Além disso, existem provavelmente milhares de dashboards compartilhados na internet para todo tipo de aplicação já prontos para usarmos em nosso ambiente.

## Prometheus no Minikube

Dada esta introdução, vamos demonstrar a instalação de um ambiente de monitoramento de um Kubernetes, neste caso não trataremos de um cluster, pois usaremos um Minikube com apenas um nó. Caso você queira testar também, os seguintes requisitos são necessários:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Helm](https://helm.sh/docs/intro/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

Não abordarei a instalação deles neste documento, então antes de seguir os próximos pontos, prepare-os de forma adequada em sua estação de trabalho.

Caso você instale o Kubernetes em um servidor diferente remoto, é importante que você execute os comandos de "port-forward" direto de sua estação. Caso isto não seja possível, você terá que customizar estes comandos para conseguir acessar os recursos de seu cluster ou prover outros métodos de acesso.

## Iniciando o Minikube

O Minikube, em meu caso, usará o Docker de minha estação para criar nosso ambiente de testes. Em alguns casos ele pode usar VM's em um hypervisor ou coisas parecidas. De qualquer modo, ele irá tratar de preparar a integração entre nosso kubectl local e o cluster K8S.

Então vamos lá, para iniciar o ambiente execute:

```
minikube start

```

![Minikube_start](img/000_minikube_start.png)

Com nosso ambiente iniciado, podemos verificar se tudo está OK rodando alguns comandos:

```
kubectl get nodes
```

Que nos retornará uma lista com os nodes de nosso cluster.

```
kubectl get pods -A
```

Que nos retornará os pods de todos os Namespaces.

Algo parecido com isso será retornado...


![ok](img/001_ok_commands.png)

## Prometheus Chart

Para simplificar a instalação de ambientes de monitoramento de clusters Kubernetes usando Prometheus, um [Chart](https://github.com/prometheus-community/helm-charts/) foi criado.

Com ele, iremos implementar toda a infraestrutura necessária para uma instalação padrão do Prometheus, Grafana, Node Exporter, Metric Server e Alert Manager. O ambiente ainda não estará pronto para produção, porém será possível testar praticamente todas as características do ambiente.

Começaremos criando um namespace para abrigar todos os objetos criados pelo Operator:

```
kubectl create namespace prometheus
```
Após isso, adicionaremos o repositório do chart em nosso ambiente e o instalaremos no namespace que criamos.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```
helm install prometheus prometheus-community/kube-prometheus-stack --namespace prometheus

```

Algo assim deve ser retornado, eu removi os Warnings de API’s descontinuadas para facilitar!

![ok](img/002_helm_install.png)

O processo de instalação demora alguns minutos. Podemos usar um comando para verificar a instalação. Ao final o ambiente deve ficar da seguinte forma:

```
kubectl get pods -n prometheus

```

![ok](img/003_helm_ok.png)

De maneira simplificada, cada um desses pods tem a seguinte finalidade: 
 
 - Alertmanager: Gerenciador de alertas do ambiente.
 - Grafana: Frontend onde veremos os gráficos.
 - Prometheus-Operator: Gerenciador da instalação.
 - Kube-State-Metrics: Expõe as métricas dos recursos internos do Kubernetes
 - Prometheus: Quem irá capturar e guardar as métricas.
 - Node-Exporter: Expõe as métricas dos servidores.

## Instalação pronta, e agora?

Com a finalização da instalação do Prometheus (e outros recursos) em nosso cluster, podemos acessá-lo com os seguintes comandos:

```
kubectl port-forward -n prometheus service/prometheus-operated 9090:9090
```

Após isto, podemos acessar no navegador de nossa estação de trabalho em: http://localhost:9090 e verificar a interface web do Prometheus!

![ok](img/005_prometheus_ui.png)

Certo, tudo certo com o Prometheus, agora vamos acessar o Grafana. Com o seguinte comando podemos acessá-lo:

```
 kubectl port-forward -n prometheus service/prometheus-grafana 3000:80
```

Agora podemos acessar http://localhost:3000 para ter acesso ao Grafana! As credenciais de acesso são: admin/prom-operator.


![ok](img/006_grafana_ui.png)

Caso o projeto evolua (o que é bem possível) e elas sejam alteradas, é possível recuperá-las através da secret criada no ambiente:

```
kubectl get secret -n prometheus prometheus-grafana -o yaml
```
A secret guarda o usuário e a senha em base64, basta decodificar os valores para ter acesso a senha do ambiente.


![ok](img/004_secret_grafana.png)

## Dashboards

O ambiente já vem com vários dashboards prontos. Eles podem ser acessados [aqui](http://localhost:3000/dashboards).

![ok](img/007_dashboard.png)

Não trataremos disso nesse documento, mas é fácil exportar partes interessantes de um dashboard e agregá-los todos em um único dashboard de gerenciamento. O Grafana também tem funções de criar playlists de Dashboards para deixarmos rodando em um monitor separado.

O Grafana é uma aplicação bem completa. Podemos configurá-lo para usar autenticação via LDAP, configurar alertas via Slack ou Mattermost e adicionar outras fontes de dados. Além disso, existem vários plugins para customizar ainda mais o ambiente. Nesta [página](https://grafana.com/grafana/dashboards) podemos encontrar dashboards oficiais e da comunidade para download.

## Sobre clusters

Estes procedimentos podem ser utilizados em ambientes com mais de um node com poucas modificações. O Node-Exporter é implementado de forma a ter um pod iniciado em cada servidor do cluster, sendo assim, as métricas de todos os servidores serão coletadas.

Os dados serão persistidos por 10 dias, porém da forma atual, caso o pod do Prometheus reinicie, todas as métricas serão perdidas. Isso pode ser resolvido de algumas formas, porém não trataremos disso nesse documento.

Provavelmente, caso você tente executar os comandos deste documento em um cluster já em funcionamento você terá resultados muito parecidos com os executados em um ambiente de teste com Minikube, esta é a beleza dos Helm Charts e Operators do Kubernetes.

## Palavras finais

Por mais que este ambiente a princípio pareça completo, ainda faltam algumas coisas para realmente o colocarmos em produção. Será necessário prover o acesso a pessoas fora do cluster pelo menos ao Grafana. Temos que emitir certificados digitais para as URLs externas e gerenciar a persistência de dados do Prometheus, afinal não queremos perder nossas métricas.

Além disto, o ambiente está com suas definições padrões, então teremos que observar nossas necessidades para alterar coisas como tempo de retenção, formas de autenticação e canais de alertas. Isso tudo pode ser feito no decorrer do tempo, conforme aprendemos melhor como usar este ambiente.

Essas customizações podem ser feitas durante a instalação do ambiente usando o Chart do Helm. Em geral usa-se um arquivo chamado "values.yaml" em conjunto com o comando de instalação:

```
helm install prometheus prometheus-community/kube-prometheus-stack -f values.yaml --namespace prometheus

```

Este arquivo pode ser encontrado [aqui](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml) com seus valores padrões. 

Por fim, 

### Remoção do ambiente

Caso você queira remover a instalação, isso pode ser feito com o comando:


```
helm uninstall prometheus -n prometheus

```

Por algum motivo, o helm não remove alguns Custom Resources criados durante a instalação. Verifique isso caso delete o ambiente!

E o cluster de Kubernetes gerido pelo Minikube pode ser deletado com:

```
minikube delete

```

Então é isso, assim encerro este documento, onde tratamos de alguns aspectos do Prometheus no Kubernetes. Entrem em contato caso encontrem algum erro neste documento ou queiram mostrar algo interessante sobre esse ambiente!



