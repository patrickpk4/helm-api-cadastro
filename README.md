# -- Aplicação Cadastro de Produto

## Kubernetes • Helm Chart • MongoDB (Dependência) • Ingress • RBAC • NetworkPolicy • Monitoramento • Grafana • Prometheus • .NET

Este repositório contém o **Helm Chart da API de Cadastro**, totalmente
integrado com **MongoDB via dependência Helm**, além de um **sistema
completo de monitoramento com Prometheus e Grafana**, tanto da **API**
quanto do **Banco de Dados MongoDB**.

------------------------------------------------------------------------

##  Visão Geral do Projeto

Este projeto demonstra uma aplicação Kubernetes realista utilizando:

-   API escrita em **.NET**
-   Deploy via **Helm Chart**
-   **MongoDB** como dependência externa
-   Observabilidade completa com:
    -   **Prometheus**
    -   **Grafana**
    -   **Dashboards customizados**
    -   **Métricas de aplicação + métricas de banco + métricas de
        Kubernetes**
-   Recursos Kubernetes:
    -   Ingress
    -   RBAC
    -   Secrets
    -   ConfigMaps
    -   NetworkPolicy
    -   HPA
    -   Persistência (PV/PVC)
    -   Arquitetura configurável via `values.yaml`

------------------------------------------------------------------------

##  Estrutura do Repositório

    helm-api-cadastro/
    ├── api-cadastro/
    │   ├── Chart.yaml
    │   ├── values.yaml
    │   ├── templates/
    │   └── charts/
    └── README.md

------------------------------------------------------------------------

##  Dependência --- MongoDB via Helm

A API utiliza o MongoDB como dependência, instalado a partir deste
repositório:

🔗 **https://github.com/patrickpk4/mongodb**

### Habilitação da dependência

``` yaml
mongodb:
  enabled: true
```

### Declaração no `Chart.yaml`

``` yaml
dependencies:
  - name: mongodb
    version: "0.1.3"
    repository: "https://patrickpk4.github.io/mongodb/"
```

### Atualizar dependências:

``` bash
helm dependency update ./api-cadastro/
```

------------------------------------------------------------------------

##  Uso de Secrets

Você pode reaproveitar um Secret existente:

``` yaml
existingSecret:
  enabled: true
  name: meu-secret-existente
```

Ou permitir que o subchart do MongoDB crie seu próprio Secret.

------------------------------------------------------------------------

##  Exemplo de `values.yaml`

``` yaml
mongodb:
  enabled: true

  service:
    type: ClusterIP
    port: 27017

  volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
      readOnly: false

  volumeclaimtemplates:
    - metadata:
        name: mongodb-data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: mongodb-nfs
        resources:
          requests:
            storage: 1Gi

  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 10
    targetCPUUtilizationPercentage: 80
    targetMemoryUtilizationPercentage: 80

  rolebinding:
    enabled: true
  role:
    enabled: true
  networkpolicy:
    enabled: true

existingSecret:
  enabled: false
  name: ""
```

------------------------------------------------------------------------

##  Ingress

O serviço pode ser exposto via hostname:

    http://cadastro.local/swagger

Necessário adicionar ao hosts:

    127.0.0.1 api-cadastro.local

------------------------------------------------------------------------

#  Monitoramento da API --- Prometheus + Grafana

O projeto possui um dashboard completo para acompanhar o desempenho da
API:

-   Latência (P95)
-   Requisições por segundo
-   Requisições ativas
-   Uso de CPU (% dos limites)
-   Uso de memória
-   Taxa de erros 4xx e 5xx
-   Métricas avançadas do .NET (GC Generation 2 etc.)

------------------------------------------------------------------------

#  Monitoramento do Banco de Dados MongoDB --- Prometheus + Grafana

Além da API, foi implementado um dashboard completo para o MongoDB,
incluindo:

------------------------------------------------------------------------

##  **1. Operações (Leituras / Escritas / Comandos)**

Métricas da família `mongodb_ss_opcounters`:

    rate(mongodb_ss_opcounters{legacy_op_type="query"}[5m])
    rate(mongodb_ss_opcounters{legacy_op_type="insert"}[5m])
    rate(mongodb_ss_opcounters{legacy_op_type="command"}[5m])

------------------------------------------------------------------------

##  **2. Conexões Ativas do MongoDB**

    mongodb_ss_connections

------------------------------------------------------------------------

##  **3. Tráfego de Rede**

    rate(mongodb_ss_network_bytesIn[5m])
    rate(mongodb_ss_network_bytesOut[5m])

------------------------------------------------------------------------

##  **4. Uso de CPU do MongoDB (% do limite)**

    sum(rate(container_cpu_usage_seconds_total{container="mongodb"}[3m]))
    /
    sum(kube_pod_container_resource_limits{container="mongodb", resource="cpu"}) * 100

------------------------------------------------------------------------

##  **5. IO de Disco (Leituras/Escritas)**

    rate(mongodb_sys_disks_sda_reads[5m])
    rate(mongodb_sys_disks_sda_writes[5m])

------------------------------------------------------------------------

## 🧩 **6. Consumo de Memória (% do limite)**

    sum(container_memory_working_set_bytes{container="mongodb"})
    /
    sum(kube_pod_container_resource_limits{resource="memory", container="mongodb"}) * 100

------------------------------------------------------------------------

##  **7. Uptime**

    mongodb_ss_uptimeEstimate

------------------------------------------------------------------------

## **8. Saúde do ReplicaSet**

    mongodb_rs_members_health

------------------------------------------------------------------------

##  **9. Réplicas do HPA (se habilitado)**

    kube_horizontalpodautoscaler_status_current_replicas{horizontalpodautoscaler="mongodb-hpa"}

------------------------------------------------------------------------


##  Autor

**Patrick Amorim**\
GitHub: https://github.com/patrickpk4
