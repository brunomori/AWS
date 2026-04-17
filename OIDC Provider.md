# 🔐 AWS EKS - OIDC Provider (IRSA)

Guia para configurar o OIDC Provider no EKS (necessário para IRSA).

---

# 📌 1. Conceito

OIDC permite associar Service Accounts do Kubernetes a IAM Roles.

👉 Isso evita usar permissões direto nos nodes (best practice).

---

# 📌 2. Verificar se já existe

aws eks describe-cluster \
  --name meu-cluster \
  --query "cluster.identity.oidc.issuer" \
  --output text

---

# 📌 3. Criar OIDC Provider (RECOMENDADO)

Usando eksctl:

eksctl utils associate-iam-oidc-provider \
  --cluster meu-cluster \
  --approve

---

# 📌 4. Criar via AWS CLI (manual)

1. Obter issuer URL:

aws eks describe-cluster \
  --name meu-cluster \
  --query "cluster.identity.oidc.issuer" \
  --output text

2. Criar provider:

aws iam create-open-id-connect-provider \
  --url <OIDC_URL> \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 9e99a48a9960b14926bb7f3b02e22da0ecd4e7b7

---

# 📌 5. Validar

aws iam list-open-id-connect-providers

---

# 📌 6. Uso com IRSA

Permite:

- Pods acessarem AWS com segurança
- Evitar uso de credenciais estáticas
- Aplicar princípio de menor privilégio

---

# 📌 7. Boas práticas

- Sempre usar IRSA em produção
- Não usar Node Role para aplicações
- Criar roles específicas por serviço

---

# 🎯 Conclusão

OIDC é essencial para:

✔ Segurança  
✔ Boas práticas AWS  
✔ Controle fino de acesso  
