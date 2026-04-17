# 🔐 AWS EKS - Node IAM Role (Worker Nodes)

Guia prático e completo para criação e configuração da **IAM Role dos Nodes (Worker Nodes)** no EKS.

---

# 📌 1. Conceito

Os **Worker Nodes (EC2)** são responsáveis por rodar seus Pods no Kubernetes.

Para isso, eles precisam de permissões para:

* 🔗 Comunicar com o **Control Plane do EKS**
* 📦 Fazer pull de imagens do **Amazon ECR**
* 🌐 Gerenciar rede via **CNI (Container Network Interface)**
* 🧾 Registrar o node no cluster Kubernetes
* 📊 Enviar logs e métricas (opcional - CloudWatch)

👉 Sem essa role, o node **nem entra no cluster**.

---

# 🧱 2. Policies obrigatórias

As seguintes policies são **ESSENCIAIS**:

* `AmazonEKSWorkerNodePolicy` → Comunicação com o cluster
* `AmazonEC2ContainerRegistryReadOnly` → Pull de imagens do ECR
* `AmazonEKS_CNI_Policy` → Configuração de rede (pods/IPs)

---

# ⭐ 3. Policies recomendadas (opcional)

Para ambientes reais:

* `CloudWatchAgentServerPolicy` → Logs e métricas
* `AmazonSSMManagedInstanceCore` → Acesso via SSM (sem SSH)

👉 Isso evita precisar abrir SSH nos nodes (best practice).

---

# 🖥️ 4. Criar via Console (AWS)

### Caminho:

IAM → Roles → **Create Role**

### Passos:

1. Selecione: **AWS Service**
2. Escolha: **EC2**
3. Clique em **Next**
4. Adicione as policies obrigatórias
5. Nome da role:

```
eks-node-role
```

6. Clique em **Create Role**

---

# 🔐 5. Trust Relationship (confiança da role)

Essa configuração permite que instâncias EC2 assumam a role:

```json
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
```

---

# 💻 6. Criar via CLI (modo profissional)

## 📄 Criar arquivo trust-policy.json

```json
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
```

---

## 🚀 Criar Role

```bash
aws iam create-role \
  --role-name eks-node-role \
  --assume-role-policy-document file://trust-policy.json
```

---

## 🔗 Attach das policies obrigatórias

```bash
aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
```

---

## ⭐ (Opcional) Attach policies recomendadas

```bash
aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

---

# ⚙️ 7. Uso no Node Group

Ao criar seu Node Group (EKS):

👉 Selecione a role:

```
eks-node-role
```

---

# 📊 8. Resultado Esperado

Após criar o Node Group corretamente:

```bash
kubectl get nodes
```

### ✔️ Saída esperada:

```bash
ip-192-168-x-x   Ready
```

👉 Isso significa que o node:

* Conectou ao cluster ✅
* Recebeu IP via CNI ✅
* Está pronto para rodar Pods ✅

---

# 🧪 9. Teste prático

Deploy simples para validar:

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

👉 Se o Pod subir → Node está OK

---

# ⚠️ 10. Erros comuns

* ❌ Node não entra no cluster
  👉 Role incorreta ou não associada

* ❌ Pod não sobe
  👉 Problema de rede (CNI policy)

* ❌ Não baixa imagem
  👉 Falta `AmazonEC2ContainerRegistryReadOnly`

* ❌ Sem acesso ao node
  👉 Falta SSM ou SSH mal configurado

---

# 🧠 11. Boas práticas

* 🔒 Evite acesso SSH → use SSM
* 🔐 Não use permissões excessivas (least privilege)
* 📊 Ative logs com CloudWatch
* 🔁 Use Auto Scaling para nodes

---

# 🎯 Conclusão

A **Node IAM Role** é fundamental para o funcionamento do EKS:

* 🔗 Permite comunicação com o cluster
* 📦 Libera acesso ao ECR
* 🌐 Habilita rede dos Pods
* ⚙️ Torna os nodes operacionais

👉 Sem ela, **o cluster simplesmente não funciona**.
