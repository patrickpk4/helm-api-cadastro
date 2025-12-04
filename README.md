

### Kubernetes • Helm Chart • Dependência MongoDB • Ingress • RBAC • NetworkPolicy • .NET API

Este repositório contém o **Helm Chart da API de Cadastro**, que agora
utiliza o **MongoDB como dependência via Helm**, hospedado em outro
repositório.

------------------------------------------------------------------------

##  Visão Geral

Este projeto demonstra:

-   Deploy da API de Cadastro escrita em **.NET**
-   Gerenciamento via **Helm Chart**
-   Conexão com **MongoDB via dependência Helm**
-   Suporte a:
    -   **Ingress**
    -   **RBAC**
    -   **Secrets**
    -   **ConfigMaps**
    -   **NetworkPolicy**
    -   **HPA**
    -   **Valores customizáveis via `values.yaml`**

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

##  Dependência -- MongoDB

O MongoDB agora é instalado a partir deste repositório:

https://github.com/patrickpk4/mongodb

O Chart da API carrega o MongoDB automaticamente se você habilitar:

``` yaml
mongodb:
  enabled: true
```

Dependência declarada no `Chart.yaml`:

``` yaml
dependencies:
  - name: mongodb
    version: "0.1.3"
    repository: "https://patrickpk4.github.io/mongodb/"
```

Atualizar dependências:

``` bash
helm dependency update ./api-cadastro/
```

------------------------------------------------------------------------

##  Uso de Secrets

Você pode usar um secret existente:

``` yaml
existingSecret:
  enabled: true
  name: meu-secret-existente
```

Ou deixar o subchart do MongoDB criar seu próprio secret.

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

##  Instalação

``` bash
helm install api-cadastro ./api-cadastro/
```

Atualizar:

``` bash
helm upgrade api-cadastro ./api-cadastro/
```

Remover:

``` bash
helm uninstall api-cadastro
```

------------------------------------------------------------------------

##  Ingress

    http://api-cadastro.local/swagger

Adicionar no `/etc/hosts`:

    127.0.0.1 api-cadastro.local

------------------------------------------------------------------------

##  Troubleshooting

``` bash
kubectl logs -l app=api-cadastro
kubectl get events --sort-by='.lastTimestamp'
```

------------------------------------------------------------------------

##  Autor

**Patrick Amorim**\
GitHub: https://github.com/patrickpk4
