# 🔐 AWS EKS - OIDC Provider (IRSA)

Guia completo para configurar o **OIDC Provider no EKS**, etapa essencial para uso de **IRSA (IAM Roles for Service Accounts)**.

---

# 📌 1. Conceito

O **OIDC Provider** permite que o Kubernetes (EKS) se integre com o IAM da AWS.

Com isso, você pode:

* 🔐 Associar **Service Accounts** a **IAM Roles**
* 🚫 Evitar uso de permissões diretamente nos Nodes
* 🎯 Aplicar **princípio do menor privilégio (least privilege)**

👉 Isso é a base do **IRSA (IAM Roles for Service Accounts)**.

---

# 🧠 2. Por que isso é importante?

Sem OIDC:

* ❌ Pods usam permissões da Node Role (inseguro)
* ❌ Todas as aplicações compartilham as mesmas permissões

Com OIDC + IRSA:

* ✅ Cada aplicação tem sua própria role
* ✅ Segurança isolada por Pod
* ✅ Controle total de acesso

---

# 🔍 3. Verificar se já existe OIDC

```bash id="p4tq6k"
aws eks describe-cluster \
  --name meu-cluster \
  --query "cluster.identity.oidc.issuer" \
  --output text
```

### ✔️ Resultado esperado:

```bash id="c5f4o0"
https://oidc.eks.us-east-1.amazonaws.com/id/XXXXXXXXXXXX
```

👉 Se retornar vazio → OIDC ainda não foi associado.

---

# 🚀 4. Criar OIDC Provider (RECOMENDADO)

### ✅ Forma mais simples (eksctl)

```bash id="7h5w2k"
eksctl utils associate-iam-oidc-provider \
  --cluster meu-cluster \
  --approve
```

👉 Esse comando:

* Cria o OIDC automaticamente
* Faz toda a configuração necessária
* Evita erros manuais

---

# ⚙️ 5. Criar via AWS CLI (modo manual)

## 📄 Passo 1: Obter OIDC URL

```bash id="j7o4dr"
aws eks describe-cluster \
  --name meu-cluster \
  --query "cluster.identity.oidc.issuer" \
  --output text
```

---

## 📄 Passo 2: Criar provider

```bash id="h5wx7g"
aws iam create-open-id-connect-provider \
  --url <OIDC_URL> \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 9e99a48a9960b14926bb7f3b02e22da0ecd4e7b7
```

👉 Substitua:

```text
<OIDC_URL>
```

Pelo valor retornado no passo anterior.

---

# ✅ 6. Validar criação

```bash id="8n8g0u"
aws iam list-open-id-connect-providers
```

### ✔️ Esperado:

```bash id="d3mtk8"
arn:aws:iam::ACCOUNT_ID:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/XXXXXXXX
```

---

# 🔗 7. Uso com IRSA (na prática)

Após configurar OIDC, você pode:

* 🔐 Criar IAM Roles específicas para aplicações
* 🔗 Associar essas roles a Service Accounts
* 🚀 Permitir que Pods acessem serviços AWS (S3, SQS, DynamoDB, etc.)

👉 Exemplo de fluxo:

1. Criar IAM Role
2. Criar ServiceAccount com annotation
3. Deploy do Pod usando essa ServiceAccount

---

# 🧪 8. Exemplo de Service Account com IRSA

```yaml id="x2k9c1"
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT_ID:role/minha-role
```

👉 Agora qualquer Pod usando essa SA herda a role.

---

# 📊 9. Resultado Esperado

Após configurar OIDC + IRSA:

* ✅ Pods acessam AWS sem credenciais hardcoded
* ✅ Cada aplicação usa sua própria role
* ✅ Segurança muito mais granular

---

# ⚠️ 10. Erros comuns

* ❌ OIDC não associado ao cluster
* ❌ ARN da role incorreto na annotation
* ❌ Trust policy da role mal configurada
* ❌ Namespace incorreto na ServiceAccount

---

# 🧠 11. Boas práticas

* 🔐 Sempre usar IRSA em produção
* 🚫 Nunca usar Node Role para aplicações
* 🎯 Criar uma role por aplicação (ou serviço)
* 🔍 Auditar permissões regularmente

---

# 🎯 Conclusão

O **OIDC Provider** é essencial para segurança no EKS:

* 🔐 Habilita IRSA
* 🎯 Permite controle fino de acesso
* 🚀 Elimina uso de credenciais estáticas

👉 Sem OIDC, você perde o principal benefício de segurança do EKS.
