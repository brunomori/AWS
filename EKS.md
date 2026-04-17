# 🚀 AWS EKS - Guia Completo (CLI + Console)

Guia prático e direto para criação de cluster Kubernetes no Amazon EKS.

---

# 📌 1. Conceito (EKS)

EKS (Elastic Kubernetes Service) é um serviço gerenciado da AWS que permite rodar Kubernetes sem precisar gerenciar o control plane.

Arquitetura básica:

Usuário → LoadBalancer → Service → Pod

---

# 📌 2. Pré-requisitos

## Conta AWS
- Conta ativa
- Permissões IAM (Admin ou equivalente)

## Ferramentas

### AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

### kubectl
```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### eksctl
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

---

# 📌 3. IAM (IMPORTANTE)

## Role do Cluster
- AmazonEKSClusterPolicy

## Role dos Nodes
- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly
- AmazonEKS_CNI_Policy

---

# 📌 4. Criação via CLI (RECOMENDADO)

## Comando simples:
```bash
eksctl create cluster   --name meu-cluster   --region us-east-1   --nodegroup-name workers   --node-type t3.medium   --nodes 2
```

---

## Comando nível projeto real:
```bash
eksctl create cluster   --name meu-cluster   --region us-east-1   --version 1.29   --nodegroup-name workers   --node-type t3.medium   --nodes 2   --nodes-min 1   --nodes-max 3   --managed
```

---

# 📌 5. Configurar acesso

```bash
aws eks update-kubeconfig   --region us-east-1   --name meu-cluster
```

Testar:
```bash
kubectl get nodes
```

---

# 📌 6. Deploy de teste

```bash
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx   --type=LoadBalancer   --port=80

kubectl get svc
```

---

# 📌 7. Criação via Console (GUI)

## Caminho:

AWS Console → EKS → Create Cluster

## Passos:

1. Nome do cluster
2. Selecionar IAM Role
3. Escolher VPC
4. Subnets públicas + privadas
5. Endpoint: Public + Private
6. Criar cluster

Depois:

EKS → Compute → Add Node Group

- Escolher Node Role
- Tipo: t3.medium
- Quantidade: 2 nodes

---

# 📌 8. Boas práticas

- Nodes em subnets privadas
- Habilitar logs
- Usar IRSA
- Separar ambientes (dev/staging/prod)
- Usar autoscaling

---

# 📌 9. Remover cluster (evitar custo)

```bash
eksctl delete cluster --name meu-cluster --region us-east-1
```

---

# 🎯 Conclusão

✔ Cluster criado  
✔ Nodes rodando  
✔ Acesso configurado  
✔ Deploy funcionando  

---

# 🚀 Próximos passos

- Ingress Controller (ALB)
- HTTPS
- CI/CD
- Terraform
