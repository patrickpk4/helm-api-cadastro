#  README – Aplicação Cadastro de Produto  
## Kubernetes • Helm Chart • Ingress • RBAC • NetworkPolicy • Persistência • MongoDB • .NET

Este projeto demonstra uma aplicação completa e realista rodando em **Kubernetes**, utilizando:

- **Helm Chart** para empacotamento e automação do deploy  
- **Ingress** para expor a API via hostname  
- **MongoDB StatefulSet** com **Volumes Persistentes (PV/PVC)**  
- **RBAC (Role, RoleBinding, ServiceAccount)** seguindo o princípio do menor privilégio  
- **NetworkPolicies** para isolar tráfego entre namespaces  
- **HPA** para autoescalonamento  
- **Secrets & ConfigMaps**  
- **Namespaces separados** para aplicação e banco  
- **NodePort / Gateway / Ingress** dependendo do ambiente

---

#  Deploy com Helm

Instalar:

```bash
helm install cadastro-produto ./chart
```

Atualizar:

```bash
helm upgrade cadastro-produto ./chart
```

Instalar:

```bash
helm install mongodb-produto ./chart
```

Atualizar:

```bash
helm upgrade mongodb-produto ./chart
```

---

#  Acesso à Aplicação via Ingress

A API é exposta por:

```
http://cadastro.local:30080/swagger
```

Se necessário, adicione ao `/etc/hosts`:

```
127.0.0.1 cadastro.local
```

---

#  Componentes Utilizados no Cluster

##  1. Persistência – PV / PVC / StorageClass
- MongoDB utiliza **StatefulSet**
- Volume persistente provisionado via NFS (ou outro provisionador)
- PVC separado por namespace  
- Política de retenção configurada

##  2. Rede – NetworkPolicies
- Controle de acesso por namespace  
- Somente a API pode se comunicar com o MongoDB  
- Isolamento total entre outros pods  
- Permissão explícita para DNS interno

##  3. RBAC – ServiceAccount / Role / RoleBinding
- Cada namespace possui seu **ServiceAccount** dedicado  
- Roles com mínimo privilégio  
- A API só pode ler Secrets e acessar seu namespace  
- MongoDB só pode ler seu secret  
- Acesso cross-namespace apenas quando necessário

##  4. Ingress & Gateway API
- Exposição via domínio `cadastro.local`  
- Pode ser usado Ingress NGINX, Contour, Traefik ou Gateway API  
- Suporte a TLS opcional  
- Helm permite ativar ou desativar via `values.yaml`

##  5. Autoescalonamento – HPA
- API escala com base em CPU  
- MongoDB possui HPA configurado com limites seguros  
- Métricas via Metrics Server

---

## Arquitetura


A aplicação é dividida em dois namespaces:

- **api-app**: Contém a API .NET Core, seus Secrets, ServiceAccount, Role, RoleBinding e HPA.
- **data-base**: Contém o MongoDB como StatefulSet, seu Secret, Headless Service, Role, RoleBinding e HPA.

A comunicação entre os namespaces é controlada por uma **NetworkPolicy**.

---

## Estrutura do Projeto

```
catalogo-kubernetes/
├── api-app/
│   ├── catalogo.yaml
│   ├── service-catalogo.yaml
│   ├── secret-catalogo.yaml
│   ├── serviceaccount-catalogo.yaml
│   ├── role-catalogo.yaml
│   ├── rolebinding-catalogo.yaml
│   ├── hpa-catalogo.yaml
│   └── networkpolicy-catalogo.yaml
├── data-base/
│   ├── mongodb.yaml
│   ├── service-mongodb.yaml
│   ├── secret-mongodb.yaml
│   ├── serviceaccount-mongodb.yaml
│   ├── role-mongodb.yaml
│   ├── rolebinding-mongodb.yaml
│   ├── hpa-mongodb.yaml
│   └── networkpolicy-mongodb.yaml
├── storage/
│   └── storage-class-mongodb.yaml
└── README.md
```

---

## Componentes Principais

| Componente | Tecnologia | Função |
|-------------|-------------|--------|
| **API Backend** | .NET Core 6.0 | API REST do catálogo |
| **Database** | MongoDB 4.4 | Armazenamento de dados |
| **Orquestração** | Kubernetes | Orquestração de containers |
| **Armazenamento** | NFS | Persistência de dados |
| **Networking** | Calico | Network policies |

---

## Serviços

| Serviço | Tipo | Porta | Descrição |
|----------|------|-------|------------|
| **api-service** | NodePort | 80/443 | Exposição externa da aplicação |
| **mongo-service** | Headless | 27017 | Comunicação interna MongoDB |

---


## Variáveis de Ambiente

**Secret do MongoDB:**
```yaml
MONGO_INITDB_ROOT_USERNAME: mongouser
MONGO_INITDB_ROOT_PASSWORD: mongopwd
```

**Secret da Aplicação:**
```yaml
Mongo__host: mongo-service.data-base.svc.cluster.local
Mongo__port: 27017
Mongo__DataBase: admin
```

---

## Resource Limits

```yaml
# API
requests:
  memory: 128Mi
  cpu: 50m
limits:
  memory: 128Mi
  cpu: 70m

# MongoDB
requests:
  memory: 180Mi
  cpu: 150m
limits:
  memory: 256Mi
  cpu: 250m
```

---

## Health Checks

**MongoDB**
- Startup: `mongo --eval "db.adminCommand('ping')"`
- Readiness: TCP socket 27017
- Liveness: `mongo --eval "db.adminCommand('ping')"`

---

## Segurança

**RBAC**
- Service Accounts específicos por namespace  
- Roles com princípio do menor privilégio  
- ClusterRoles para acesso cross-namespace  

**Network Policies**
- Isolamento entre namespaces  
- Acesso restrito à porta do MongoDB  
- Permissão para DNS resolution  

**Secrets**
- Credenciais em Base64  
- Namespace isolation  
- Controle de acesso via RBAC  

---

## Persistência

**StorageClass NFS**
```yaml
provisioner: cluster.local/nfs-subdir-external-provisioner
server: 192.168.1.21
path: /export
reclaimPolicy: Retain
```

---

## Monitoramento

```bash
kubectl get pods -n api-app
kubectl get pods -n data-base
kubectl logs -n api-app -l app=api
kubectl logs -n data-base -l app=mongodb
kubectl get hpa -A
```

---

## Troubleshooting

```bash
kubectl get events -A --sort-by='.lastTimestamp'
kubectl run test-connection -n api-app --image=busybox --rm -it -- sh
kubectl auth can-i get secrets --as=system:serviceaccount:api-app:catalogo-service-account
kubectl rollout restart deployment/api-deployment -n api-app
```

---

## Boas Práticas Implementadas

 Multi-namespace para isolamento  
 RBAC com princípio do menor privilégio  
 Health checks completos  
 Auto-scaling horizontal  
 Persistência com NFS  
 Network policies para segurança  
 Resource limits definidos  
 Secrets management adequado  
 StatefulSet para banco de dados  
 Probes para resiliência  

---

## Autor

**Patrick Amorim**  
Projeto de estudo em Kubernetes, MongoDB e .NET Core  
GitHub: [@patrickpk4](https://github.com/patrickpk4)

---

## Licença
Projeto de estudo e uso educacional.
