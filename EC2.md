# 🖥️ AWS EC2 - Guia Prático (Console + CLI)

## 🎯 Objetivo
Aprender na prática:
- Criar Key Pair (SSH)
- Criar EC2 via Console
- Criar EC2 via CLI
- Entender cada campo importante

---

# 🔹 1. Key Pair (SSH)

## 📌 O que é?
Arquivo usado para acessar sua EC2 via SSH (sem senha).

👉 Você baixa um `.pem` e usa para conectar.

---

## 🌐 Criar via Console

1. Acesse EC2 → Key Pairs
2. Clique em **Create key pair**
3. Nome: `minha-chave`
4. Tipo: RSA
5. Formato: `.pem`
6. Download automático

⚠️ IMPORTANTE:
- Guarde esse arquivo
- Não dá pra baixar de novo

---

## ⚙️ Criar via CLI

```bash
aws ec2 create-key-pair   --key-name minha-chave   --query 'KeyMaterial'   --output text > minha-chave.pem
```

### Ajustar permissão:
```bash
chmod 400 minha-chave.pem
```

---

# 🔹 2. Launch an Instance (Console)

## 📌 Caminho
EC2 → Instances → Launch Instance

---

## ⚙️ Principais configurações

### 🏷️ Name and Tags
Nome da máquina (ex: ec2-dev)

---

### 🖥️ AMI (Sistema Operacional)
Exemplo:
- Amazon Linux
- Ubuntu

👉 Para estudo: use Amazon Linux

---

### 💻 Instance Type
Exemplo:
- t2.micro (free tier)

---

### 🔑 Key Pair
Selecione a chave criada:
- `minha-chave`

---

### 🌐 Network Settings

#### VPC
Escolha sua VPC

#### Subnet
- Pública → acesso via internet
- Privada → sem acesso direto

#### Auto-assign public IP
✔ ENABLED (para acesso SSH)

#### Security Group

Criar regra:

| Tipo | Porta | Origem |
|------|------|--------|
| SSH  | 22   | 0.0.0.0/0 (ou seu IP) |

⚠️ Melhor prática:
- usar seu IP ao invés de 0.0.0.0/0

---

### 💾 Storage
Default já serve:
- 8GB (gp2/gp3)

---

### 🚀 Launch
Clique em **Launch Instance**

---

## ✅ Resultado esperado

- Instance state: **running**
- IP público disponível
- Status checks: 2/2

---

# 🔹 3. Conectar via SSH

```bash
ssh -i minha-chave.pem ec2-user@IP_PUBLICO
```

Para Ubuntu:
```bash
ssh -i minha-chave.pem ubuntu@IP_PUBLICO
```

---

# 🔹 4. Criar EC2 via CLI

## 🧱 Exemplo completo

```bash
aws ec2 run-instances   --image-id ami-123456   --instance-type t2.micro   --key-name minha-chave   --security-group-ids sg-123456   --subnet-id subnet-123456   --associate-public-ip-address
```

---

## 📌 Parâmetros importantes

| Parâmetro | Explicação |
|----------|-----------|
| image-id | AMI |
| instance-type | Tipo da máquina |
| key-name | Chave SSH |
| subnet-id | Subnet |
| sg | Regras de firewall |
| public IP | acesso externo |

---

# 🔹 5. Segurança (MUITO IMPORTANTE)

## ❌ Evite:
- SSH aberto para 0.0.0.0/0 em produção

## ✔ Use:
- Seu IP fixo
- Bastion Host (avançado)

---

# 🔹 6. Troubleshooting

## ❌ Erro comum: Permission denied

✔ Resolver:
```bash
chmod 400 minha-chave.pem
```

---

## ❌ Timeout SSH

✔ Verificar:
- Security Group liberando porta 22
- EC2 com IP público
- Subnet pública

---

# 🔹 7. Checklist Final

✔ EC2 rodando  
✔ IP público  
✔ Security Group OK  
✔ Key Pair criada  
✔ Permissão do .pem correta  

---

# 🧪 Exercício prático

1. Criar Key Pair
2. Subir EC2 t2.micro
3. Liberar SSH
4. Conectar via terminal

---

# 🔥 Conclusão

Se tudo estiver certo:
- Você consegue acessar sua EC2 via SSH
- Já está pronto para usar com Kubernetes, Docker e AWS

