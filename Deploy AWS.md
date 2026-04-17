# 🚀 Deploy AWS Load Balancer Controller no EKS

Guia prático e direto para instalar e configurar o **AWS Load Balancer Controller** em um cluster EKS.

---

# 📌 1. Conceito

O AWS Load Balancer Controller permite:

* Criar **Application Load Balancers (ALB)** automaticamente
* Integrar com **Ingress do Kubernetes**
* Expor aplicações HTTP/HTTPS de forma gerenciada

👉 Ele substitui o uso manual de ELB clássico.

---

# 🧱 2. Pré-requisitos

* Cluster EKS funcionando
* kubectl configurado
* AWS CLI configurado
* Helm instalado

---

# 🔐 3. Criar IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

# 🔗 4. Criar Service Account (IRSA)

```bash
eksctl create iamserviceaccount \
  --cluster=SEU_CLUSTER \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name=AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::SEU_ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

---

# 📦 5. Instalar via Helm

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=SEU_CLUSTER \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

---

# ✅ 6. Verificar instalação

```bash
kubectl get pods -n kube-system
```

### ✔️ Esperado:

```bash
aws-load-balancer-controller-xxxxx   Running
```

---

# 🌐 7. Exemplo de Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

### Aplicar:

```bash
kubectl apply -f ingress.yaml
```

---

# 📊 8. Resultado Esperado

Após aplicar o Ingress:

* ✅ Um **ALB será criado automaticamente**
* ✅ Aparecerá no console da AWS
* ✅ Um **DNS público será gerado**

### Verificar:

```bash
kubectl get ingress
```

### Saída esperada:

```bash
ADDRESS: xxxxxx.us-east-1.elb.amazonaws.com
```

---

# 🧠 9. Boas práticas

* 🔒 Use HTTPS com ACM
* 🔐 Configure Security Groups corretamente
* 🔀 Use **path-based routing**
* 🚫 Evite expor serviços desnecessários

---

# 🧪 10. Teste

```bash
curl http://SEU_DNS
```

👉 Deve retornar sua aplicação.

---

# ⚠️ Problemas comuns

* ❌ Falha de permissão IAM
* ❌ ServiceAccount não criada corretamente
* ❌ ClusterName incorreto
* ❌ Ingress sem annotation `alb`

---

# 🎯 Conclusão

Agora você tem:

* ✅ Ingress funcionando
* ✅ ALB automático
* ✅ Integração AWS + Kubernetes

👉 Base real de projeto de produção.
