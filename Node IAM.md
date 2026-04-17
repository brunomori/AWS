# 🔐 AWS EKS - Node IAM Role

Guia prático para criação da IAM Role dos Nodes (Worker Nodes) no EKS.

---

# 📌 1. Conceito

Os Nodes (EC2) precisam de permissões para:

- Comunicar com o cluster
- Fazer pull de imagens (ECR)
- Configurar rede (CNI)
- Registrar no Kubernetes

---

# 📌 2. Policies obrigatórias

- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly
- AmazonEKS_CNI_Policy

---

# 📌 3. Criar via Console

Caminho:

IAM → Roles → Create Role

Passos:

1. AWS Service
2. EC2
3. Adicionar policies obrigatórias
4. Nome: eks-node-role
5. Create

---

# 📌 4. Trust Relationship

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

---

# 📌 5. Criar via CLI

## trust-policy.json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

---

## Criar Role

aws iam create-role \
  --role-name eks-node-role \
  --assume-role-policy-document file://trust-policy.json

---

## Attach Policies

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy

---

# 📌 6. Uso no Node Group

Selecionar: eks-node-role

---

# 📌 7. Erros comuns

Node não sobe → Role errada  
Imagem não baixa → falta ECR policy  
Problema rede → falta CNI policy  

---

# 🎯 Conclusão

Role essencial para funcionamento dos nodes no EKS.
