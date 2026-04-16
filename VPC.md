# 🌐 AWS Networking COMPLETO - VPC, Subnets, IGW, NAT Gateway (Console + CLI)

## 🎯 Objetivo
Guia completo e PRÁTICO com:
- Internet Gateway
- NAT Gateway
- Subnets públicas e privadas
- Route Tables
- Tags para Load Balancer
- Console + CLI

---

# 🔹 1. Arquitetura (VISÃO GERAL)


<img width="1317" height="745" alt="image" src="https://github.com/user-attachments/assets/daa0477c-6ada-4288-92e3-e5f5e34dd60d" />




Internet → IGW → Subnet Pública → NAT Gateway → Subnet Privada

---

# 🔹 2. Internet Gateway (IGW)

## 📌 Função
Permite acesso da VPC à internet.

---

## 🌐 Console
EC2 → Internet Gateways → Create  
Depois: Attach to VPC

---

## ⚙️ CLI

```bash
aws ec2 create-internet-gateway

aws ec2 attach-internet-gateway     --internet-gateway-id igw-123     --vpc-id vpc-123
```

---

# 🔹 3. Subnets

## 🌍 Pública
- Tem rota → IGW
- Tem IP público

## 🔒 Privada
- NÃO tem acesso direto à internet
- Usa NAT Gateway

---

# 🔹 4. Route Tables

## 📌 Pública (SEU CASO)

| Destination | Target |
|------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | IGW |

---

## 📌 Privada

| Destination | Target |
|------------|--------|
| 0.0.0.0/0 | NAT Gateway |

---

# 🔹 5. NAT Gateway

## 📌 O que é?

Serviço que permite que instâncias **privadas saiam para internet**, sem receber conexão externa.

👉 Exemplo:
- EC2 privada baixa pacote → OK
- Internet acessa EC2 → NÃO

---

# 🔹 6. Criar NAT Gateway (CONSOLE)

## Caminho:
VPC → NAT Gateways → Create NAT Gateway

---

## ⚙️ Configurações

### Name
```
comunidadedevops-ngw-1a
```

---

### Availability Mode

- **Regional (novo)** → recomendado
- Zonal → controle manual

---

### VPC
Selecionar sua VPC

---

### Subnet
⚠️ IMPORTANTE:
- Deve ser **SUBNET PÚBLICA**

---

### Connectivity Type

- Public (mais comum)
- Private (casos avançados)

---

### Elastic IP (EIP)

#### Automático (RECOMENDADO)
AWS gerencia

#### Manual
Você cria:

```bash
aws ec2 allocate-address
```

---

## ✅ Resultado esperado

- NAT Gateway criado
- Status: Available

---

# 🔹 7. Criar NAT Gateway (CLI)

```bash
aws ec2 create-nat-gateway     --subnet-id subnet-publica     --allocation-id eipalloc-123
```

---

# 🔹 8. Conectar NAT Gateway à Route Table

## 📌 Subnet PRIVADA

Adicionar rota:

```bash
aws ec2 create-route     --route-table-id rtb-privada     --destination-cidr-block 0.0.0.0/0     --nat-gateway-id nat-123
```

---

## 🔁 Fluxo de comunicação

### 🌍 Subnet Pública
EC2 → IGW → Internet

---

### 🔒 Subnet Privada
EC2 → NAT → IGW → Internet

---

# 🔹 9. Dependências (ENTENDA ISSO)

## Ordem correta:

1. Criar VPC  
2. Criar Subnets  
3. Criar IGW  
4. Attach IGW  
5. Criar Subnet Pública  
6. Criar EIP  
7. Criar NAT Gateway (na subnet pública)  
8. Criar Route Table privada  
9. Apontar rota → NAT  

---

# 🔹 10. Relação entre componentes

| Componente | Se conecta com |
|----------|---------------|
| IGW | VPC |
| NAT Gateway | Subnet pública + EIP |
| Route Table | Subnet |
| Subnet privada | NAT |
| Subnet pública | IGW |

---

# 🔹 11. Tags para Load Balancer

## Pública
```
kubernetes.io/role/elb = 1
```

## Privada
```
kubernetes.io/role/internal-elb = 1
```

---

# 🔹 12. Checklist FINAL

✔ IGW criado e anexado  
✔ Subnet pública com rota para IGW  
✔ NAT Gateway criado em subnet pública  
✔ Subnet privada com rota para NAT  
✔ EIP associado  
✔ Tags configuradas  

---

# 🧪 Exercício prático

1. Criar subnet pública e privada  
2. Criar IGW  
3. Criar NAT Gateway  
4. Configurar route tables  
5. Testar acesso da EC2 privada  

---

# 🔥 Conclusão

Se tudo estiver certo:

✔ Subnet pública acessa internet direto  
✔ Subnet privada acessa via NAT  
✔ Infra pronta para EKS / produção  
