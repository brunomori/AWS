# 🌐 AWS Networking - Guia Prático (Subnets, IGW, Route Tables e Tags)

## 🎯 Objetivo
Este guia explica de forma prática:
- Internet Gateway (IGW)
- Route Tables
- Subnets (pública vs privada)
- Tags para AWS Load Balancer Controller

---

# 🔹 1. Internet Gateway (IGW)

## 📌 O que é?
O **Internet Gateway** permite comunicação entre sua VPC e a internet.

👉 Sem IGW = recursos NÃO acessam a internet.

---

## ⚙️ Criar IGW (CLI)

```bash
aws ec2 create-internet-gateway
```

### Resultado esperado:
- Retorna um `InternetGatewayId` (ex: igw-123456)

---

## 🔗 Associar IGW à VPC

```bash
aws ec2 attach-internet-gateway   --internet-gateway-id igw-123456   --vpc-id vpc-123456
```

---

# 🔹 2. Route Tables (Tabela de Roteamento)

## 📌 O que é?
Define **para onde o tráfego da rede vai**.

---

## 🧠 Exemplo prático (SEU CASO)

| Destination | Target        | Explicação |
|------------|--------------|-----------|
| 10.0.0.0/16 | local        | Comunicação interna na VPC |
| 0.0.0.0/0   | IGW          | Saída para internet |

---

## ⚙️ Criar Route Table

```bash
aws ec2 create-route-table --vpc-id vpc-123456
```

---

## ➕ Adicionar rota para internet

```bash
aws ec2 create-route   --route-table-id rtb-123456   --destination-cidr-block 0.0.0.0/0   --gateway-id igw-123456
```

---

## 🔗 Associar Route Table à Subnet

```bash
aws ec2 associate-route-table   --subnet-id subnet-123456   --route-table-id rtb-123456
```

---

# 🔹 3. Subnet Pública vs Privada

## 🌍 Subnet Pública

✔ Tem rota para internet:
```
0.0.0.0/0 → IGW
```

✔ Auto-assign IP público: **ATIVADO**

👉 Permite acesso externo (ex: Load Balancer)

---

## 🔒 Subnet Privada

✔ NÃO tem rota direta para IGW

👉 Usada para:
- Banco de dados
- Backend
- Pods internos

---

# 🔹 4. Auto-assign Public IP

## 📌 O que é?
Configuração que define se instâncias recebem IP público automaticamente.

### ✔ Ativar:
- Subnets públicas

### ❌ Desativar:
- Subnets privadas

---

# 🔹 5. Tags para AWS Load Balancer Controller

## 📌 POR QUE isso é importante?

O controller usa **tags** para descobrir automaticamente:
- Subnets públicas
- Subnets privadas

---

## 🏷️ PADRÃO OBRIGATÓRIO

### 🌍 Subnet Pública

```bash
Key: kubernetes.io/role/elb
Value: 1
```

---

### 🔒 Subnet Privada

```bash
Key: kubernetes.io/role/internal-elb
Value: 1
```

---

## 🧠 Tag de Cluster (IMPORTANTE)

```bash
Key: kubernetes.io/cluster/NOME-DO-CLUSTER
Value: shared
```

ou

```bash
Value: owned
```

---

## ⚙️ Aplicar TAG via CLI

```bash
aws ec2 create-tags   --resources subnet-123456   --tags Key=kubernetes.io/role/elb,Value=1
```

---

# 🔹 6. Resumo do Fluxo Completo

1. Criar VPC
2. Criar Subnets
3. Criar Internet Gateway
4. Attach IGW à VPC
5. Criar Route Table
6. Adicionar rota:
   - `0.0.0.0/0 → IGW`
7. Associar Route Table à Subnet pública
8. Ativar Auto-assign IP
9. Adicionar TAGS nas subnets

---

# 🔹 7. Checklist Rápido (ERROS COMUNS)

❌ Esqueceu IGW  
❌ Subnet sem rota para internet  
❌ Route Table não associada  
❌ Tags erradas ou ausentes  
❌ Auto-assign IP desativado  

---

# 🔥 Conclusão

Se tudo estiver correto:
- Load Balancer será criado automaticamente
- Pods terão acesso correto
- Infra estará padrão mercado (EKS ready)

---

# 🧪 Exercício prático

1. Crie uma subnet pública
2. Configure rota para IGW
3. Ative IP público
4. Adicione tag:

```bash
kubernetes.io/role/elb=1
```

👉 Depois tente subir um Load Balancer no Kubernetes.
