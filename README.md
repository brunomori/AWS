# ☁️ AWS | Bruno Mori

> Referência de consulta rápida para o dia a dia na AWS.
> Comandos reais via AWS CLI, troubleshooting e notas SRE.
> Foco: IAM · EC2 · VPC · EKS · ECS · ECR · CloudWatch

---

## 📑 Índice

| # | Tema |
|---|------|
| 1 | [AWS CLI — Setup e Configuração](#-aws-cli--setup-e-configuração) |
| 2 | [IAM — Identidade e Acesso](#-iam--identidade-e-acesso) |
| 3 | [EC2 — Instâncias](#-ec2--instâncias) |
| 4 | [VPC — Rede](#-vpc--rede) |
| 5 | [ECR — Registry de Containers](#-ecr--registry-de-containers) |
| 6 | [ECS — Containers sem K8s](#-ecs--containers-sem-k8s) |
| 7 | [EKS — Kubernetes Gerenciado](#-eks--kubernetes-gerenciado) |
| 8 | [CloudWatch — Observabilidade](#-cloudwatch--observabilidade) |
| 9 | [ALB / ELB — Load Balancer](#-alb--elb--load-balancer) |
| 10 | [S3 — Armazenamento de Objetos](#-s3--armazenamento-de-objetos) |
| 11 | [Troubleshooting Rápido](#-troubleshooting-rápido) |
| 12 | [Flags e Truques Úteis](#-flags-e-truques-úteis) |

---

## ⚙️ AWS CLI — Setup e Configuração

> Equivalente ao `kubectl` no Kubernetes. Instale e configure antes de qualquer coisa.

```bash
# Verificar versão
aws --version

# Configurar credenciais (cria ~/.aws/credentials e ~/.aws/config)
aws configure
# Pede: AWS Access Key ID, Secret Access Key, Region, Output format

# Verificar quem você é (usuário/role logado)
aws sts get-caller-identity

# Listar perfis configurados
cat ~/.aws/config

# Usar perfil específico
aws s3 ls --profile meu-perfil

# Definir região em um comando específico
aws ec2 describe-instances --region us-east-1

# Definir região padrão via variável de ambiente
export AWS_DEFAULT_REGION=us-east-1
export AWS_PROFILE=meu-perfil
```

> 💡 Em empresas normalmente usam roles assumidas via SSO ou `aws sts assume-role`. Confirme com o time qual o método padrão.

---

## 🔐 IAM — Identidade e Acesso

> Controla QUEM pode fazer O QUÊ em quais recursos. Entender IAM é fundamento de tudo na AWS.

### Conceitos rápidos

| Conceito | O que é |
|---|---|
| **User** | Pessoa física com credenciais |
| **Group** | Conjunto de users com mesmas permissões |
| **Role** | Identidade assumível por serviços/apps (sem senha) |
| **Policy** | Documento JSON que define permissões |
| **STS** | Serviço que emite credenciais temporárias |

```bash
# Usuários
aws iam list-users
aws iam get-user --user-name <nome>
aws iam create-user --user-name <nome>
aws iam delete-user --user-name <nome>

# Grupos
aws iam list-groups
aws iam add-user-to-group --user-name <user> --group-name <grupo>
aws iam remove-user-from-group --user-name <user> --group-name <grupo>

# Roles
aws iam list-roles
aws iam get-role --role-name <nome>
aws iam create-role --role-name <nome> --assume-role-policy-document file://trust-policy.json

# Policies
aws iam list-attached-user-policies --user-name <nome>
aws iam list-attached-role-policies --role-name <nome>
aws iam attach-role-policy --role-name <nome> --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam detach-role-policy --role-name <nome> --policy-arn <arn>

# Assumir uma role (STS)
aws sts assume-role \
  --role-arn arn:aws:iam::<account-id>:role/<role-name> \
  --role-session-name minha-sessao

# Verificar permissões de uma action específica (simulation)
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::<account>:user/<user> \
  --action-names s3:GetObject
```

### Trust Policy mínima (para Role assumida por EC2)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

> ⚠️ **Regra de ouro IAM:** sempre use **least privilege** — dê só a permissão mínima necessária. Nunca use `AdministratorAccess` em roles de aplicação.

---

## 💻 EC2 — Instâncias

> Servidores virtuais na AWS. Em SRE você vai mais inspecionar e conectar do que criar.

```bash
# Listar instâncias
aws ec2 describe-instances
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' --output table

# Filtrar instâncias por tag ou estado
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances --filters "Name=tag:Name,Values=meu-servidor"

# Iniciar / parar / reiniciar
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Conectar via SSM (sem precisar de chave SSH — padrão corporativo)
aws ssm start-session --target i-1234567890abcdef0

# Conectar via SSH (quando tem key pair)
ssh -i chave.pem ec2-user@<IP-publico>
ssh -i chave.pem ubuntu@<IP-publico>     # depende da AMI

# Ver logs de inicialização da instância
aws ec2 get-console-output --instance-id i-1234567890abcdef0

# Security Groups
aws ec2 describe-security-groups --group-ids sg-1234567890
aws ec2 describe-security-groups --filters "Name=group-name,Values=meu-sg"

# Key Pairs
aws ec2 describe-key-pairs
```

### Tipos de instância — referência rápida

| Família | Para quê |
|---|---|
| `t3`, `t4g` | Uso geral / dev / baixo custo |
| `m6i`, `m7i` | Uso geral balanceado (produção) |
| `c6i`, `c7i` | CPU intensivo |
| `r6i`, `r7i` | Memória intensiva (bancos, cache) |

> 💡 Em SRE você raramente cria instâncias manualmente — isso fica no Terraform/IaC. Você vai mais **investigar, conectar e monitorar**.

---

## 🌐 VPC — Rede

> Virtual Private Cloud — sua rede isolada dentro da AWS.

### Conceitos rápidos

| Conceito | O que é |
|---|---|
| **VPC** | Rede virtual isolada |
| **Subnet** | Subdivisão da VPC (pública ou privada) |
| **IGW** | Internet Gateway — saída para internet |
| **NAT Gateway** | Permite saída de subnets privadas para internet |
| **Route Table** | Tabela de rotas da subnet |
| **Security Group** | Firewall por instância (stateful) |
| **NACL** | Firewall por subnet (stateless) |

```bash
# VPCs
aws ec2 describe-vpcs
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true"

# Subnets
aws ec2 describe-subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-12345"
aws ec2 describe-subnets --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,Tags[?Key==`Name`].Value|[0]]' --output table

# Security Groups
aws ec2 describe-security-groups
aws ec2 describe-security-groups --filters "Name=vpc-id,Values=vpc-12345"

# Route Tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-12345"

# Internet Gateways
aws ec2 describe-internet-gateways

# NAT Gateways
aws ec2 describe-nat-gateways
```

### Diagrama mental de uma VPC típica

```
VPC (10.0.0.0/16)
├── Subnet Pública  (10.0.1.0/24) → Route Table → IGW → Internet
│     └── ALB, Bastion Host
└── Subnet Privada  (10.0.2.0/24) → Route Table → NAT Gateway → Internet
      └── EC2, ECS Tasks, RDS
```

> 💡 **Regra SRE:** aplicações e bancos ficam em **subnets privadas**. Só o Load Balancer fica na subnet pública.

---

## 📦 ECR — Registry de Containers

> Equivalente ao Docker Hub, mas dentro da AWS. Onde ficam as imagens que o ECS/EKS usa.

```bash
# Listar repositórios
aws ecr describe-repositories
aws ecr describe-repositories --query 'repositories[*].[repositoryName,repositoryUri]' --output table

# Login no ECR (para fazer push/pull de imagens)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Criar repositório
aws ecr create-repository --repository-name minha-app

# Listar imagens de um repositório
aws ecr list-images --repository-name minha-app
aws ecr describe-images --repository-name minha-app

# Push de imagem (fluxo completo)
docker build -t minha-app .
docker tag minha-app:latest <account-id>.dkr.ecr.us-east-1.amazonaws.com/minha-app:latest
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/minha-app:latest

# Deletar imagem
aws ecr batch-delete-image \
  --repository-name minha-app \
  --image-ids imageTag=latest
```

> ⚠️ O login do ECR expira em 12h. Em pipelines CI/CD o login é refeito automaticamente. Se der `unauthorized`, refaça o login.

---

## 🐳 ECS — Containers sem K8s

> Orquestrador de containers da AWS. Mais simples que Kubernetes — sem nodes para gerenciar no modo Fargate.

### Conceitos rápidos

| Conceito | Equivalente K8s | O que é |
|---|---|---|
| **Cluster** | Cluster | Agrupamento lógico |
| **Task Definition** | Pod spec / Deployment spec | Template do container |
| **Task** | Pod | Instância rodando |
| **Service** | Deployment + Service | Mantém N tasks rodando |
| **Fargate** | Managed nodes | Sem gerenciar EC2 |

```bash
# Clusters
aws ecs list-clusters
aws ecs describe-clusters --clusters meu-cluster

# Services
aws ecs list-services --cluster meu-cluster
aws ecs describe-services --cluster meu-cluster --services meu-service

# Tasks (pods)
aws ecs list-tasks --cluster meu-cluster
aws ecs list-tasks --cluster meu-cluster --service-name meu-service
aws ecs describe-tasks --cluster meu-cluster --tasks <task-arn>

# Task Definitions
aws ecs list-task-definitions
aws ecs describe-task-definition --task-definition minha-app:3

# Forçar novo deploy (equivalente ao rollout restart do K8s)
aws ecs update-service \
  --cluster meu-cluster \
  --service meu-service \
  --force-new-deployment

# Escalar service
aws ecs update-service \
  --cluster meu-cluster \
  --service meu-service \
  --desired-count 3

# Entrar em uma task Fargate via ECS Exec (equivalente ao kubectl exec)
aws ecs execute-command \
  --cluster meu-cluster \
  --task <task-arn> \
  --container meu-container \
  --interactive \
  --command "/bin/sh"

# Parar uma task específica
aws ecs stop-task --cluster meu-cluster --task <task-arn>

# Ver logs de uma task (redireciona para CloudWatch)
# Veja a seção CloudWatch
```

> 💡 `--force-new-deployment` é o `rollout restart` do ECS — recria as tasks sem alterar a task definition. Use quando precisar forçar reload de variáveis ou testar nova imagem com a mesma tag.

---

## ☸️ EKS — Kubernetes Gerenciado

> Kubernetes na AWS. O control plane é gerenciado pela AWS, você gerencia os worker nodes (ou usa Fargate).

```bash
# Configurar kubeconfig para um cluster EKS
aws eks update-kubeconfig --region us-east-1 --name meu-cluster
# Depois disso, todos os comandos kubectl funcionam normalmente

# Listar clusters
aws eks list-clusters

# Detalhes do cluster
aws eks describe-cluster --name meu-cluster

# Node Groups (worker nodes gerenciados)
aws eks list-nodegroups --cluster-name meu-cluster
aws eks describe-nodegroup --cluster-name meu-cluster --nodegroup-name meu-nodegroup

# Escalar node group
aws eks update-nodegroup-config \
  --cluster-name meu-cluster \
  --nodegroup-name meu-nodegroup \
  --scaling-config minSize=2,maxSize=10,desiredSize=3

# Add-ons do cluster (CoreDNS, kube-proxy, vpc-cni, etc.)
aws eks list-addons --cluster-name meu-cluster
aws eks describe-addon --cluster-name meu-cluster --addon-name coredns

# Fargate profiles
aws eks list-fargate-profiles --cluster-name meu-cluster
```

### Após configurar o kubeconfig, use kubectl normalmente

```bash
kubectl get nodes                         # ver worker nodes do EKS
kubectl get nodes -o wide                 # com tipo de instância e IP
kubectl describe node <nome>              # recursos disponíveis no node
kubectl top nodes                         # consumo de CPU/memória por node
```

> 💡 No EKS cada node é uma EC2. Se um pod não sobe por falta de recurso, pode ser necessário escalar o node group.

---

## 📊 CloudWatch — Observabilidade

> Logs, métricas e alertas da AWS. Equivalente ao Prometheus + Grafana + Loki juntos.

```bash
# ===== LOGS =====

# Listar log groups
aws logs describe-log-groups
aws logs describe-log-groups --log-group-name-prefix /ecs/

# Listar log streams dentro de um group
aws logs describe-log-streams --log-group-name /ecs/meu-service
aws logs describe-log-streams \
  --log-group-name /ecs/meu-service \
  --order-by LastEventTime \
  --descending

# Ver logs (últimas mensagens)
aws logs get-log-events \
  --log-group-name /ecs/meu-service \
  --log-stream-name <stream-name> \
  --limit 50

# Buscar logs com filtro (equivalente ao grep nos logs)
aws logs filter-log-events \
  --log-group-name /ecs/meu-service \
  --filter-pattern "ERROR"

aws logs filter-log-events \
  --log-group-name /ecs/meu-service \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s000)

# Seguir logs em tempo real (tail -f)
aws logs tail /ecs/meu-service --follow
aws logs tail /ecs/meu-service --follow --filter-pattern "ERROR"

# ===== MÉTRICAS =====

# Ver métricas disponíveis de um namespace
aws cloudwatch list-metrics --namespace AWS/ECS
aws cloudwatch list-metrics --namespace AWS/EC2

# Pegar valor de uma métrica
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name CPUUtilization \
  --dimensions Name=ClusterName,Value=meu-cluster Name=ServiceName,Value=meu-service \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 300 \
  --statistics Average

# ===== ALARMES =====

# Listar alarmes
aws cloudwatch describe-alarms
aws cloudwatch describe-alarms --alarm-names meu-alarme
aws cloudwatch describe-alarms --state-value ALARM   # só os que estão disparados

# Ver histórico de um alarme
aws cloudwatch describe-alarm-history --alarm-name meu-alarme
```

### Log Groups padrão por serviço

| Serviço | Padrão de Log Group |
|---|---|
| ECS | `/ecs/<nome-do-service>` |
| EKS (sistema) | `/aws/eks/<cluster>/cluster` |
| EC2 (com agent) | `/var/log/messages` ou customizado |
| Lambda | `/aws/lambda/<nome-da-function>` |
| ALB | Configurável em bucket S3 |

> 💡 **Fluxo de investigação SRE:** Alarme dispara → CloudWatch Logs → `filter-log-events` com `ERROR` → identifica task/pod com problema → `describe-tasks` ou `kubectl describe pod`.

---

## ⚖️ ALB / ELB — Load Balancer

> Distribui tráfego entre instâncias/containers. O ALB (Application) é o mais comum para HTTP/HTTPS.

```bash
# Listar load balancers
aws elbv2 describe-load-balancers
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[*].[LoadBalancerName,DNSName,State.Code]' \
  --output table

# Target Groups (grupos de destino — EC2, IPs, Lambda)
aws elbv2 describe-target-groups
aws elbv2 describe-target-groups --load-balancer-arn <arn>

# Saúde dos targets (equivalente ao kubectl get endpoints)
aws elbv2 describe-target-health --target-group-arn <arn>

# Listeners (portas e regras)
aws elbv2 describe-listeners --load-balancer-arn <arn>

# Regras de um listener
aws elbv2 describe-rules --listener-arn <arn>
```

### Status dos targets

| Status | Significado |
|---|---|
| `healthy` | Target respondendo normalmente |
| `unhealthy` | Health check falhando — investigar logs |
| `draining` | Removendo conexões antes de desregistrar |
| `unused` | Target registrado mas não em uso |

> ⚠️ Target `unhealthy` no ALB = ReadinessProbe falhando no K8s. Vá nos logs do container/task para investigar.

---

## 🪣 S3 — Armazenamento de Objetos

> Armazenamento de arquivos, backups, logs, artifacts de deploy.

```bash
# Listar buckets
aws s3 ls
aws s3api list-buckets

# Listar arquivos de um bucket
aws s3 ls s3://meu-bucket
aws s3 ls s3://meu-bucket/pasta/ --recursive

# Copiar arquivos
aws s3 cp arquivo.txt s3://meu-bucket/
aws s3 cp s3://meu-bucket/arquivo.txt ./

# Sincronizar diretório
aws s3 sync ./pasta s3://meu-bucket/pasta/
aws s3 sync s3://meu-bucket/pasta/ ./pasta

# Deletar
aws s3 rm s3://meu-bucket/arquivo.txt
aws s3 rm s3://meu-bucket/pasta/ --recursive

# Ver política de um bucket
aws s3api get-bucket-policy --bucket meu-bucket

# Verificar bloqueio público
aws s3api get-public-access-block --bucket meu-bucket
```

> 💡 Em SRE o S3 aparece muito como destino de logs (ALB, CloudTrail), artifacts de CI/CD e backup de configurações.

---

## 🔍 Troubleshooting Rápido

### Container/Task ECS não sobe

```bash
# 1. Ver status do service
aws ecs describe-services --cluster <cluster> --services <service>
# Olhar: runningCount vs desiredCount, eventos em "events"

# 2. Ver tasks recentes (incluindo as que falharam)
aws ecs list-tasks --cluster <cluster> --service-name <service> --desired-status STOPPED
aws ecs describe-tasks --cluster <cluster> --tasks <task-arn>
# Olhar: stoppedReason, containers[].reason

# 3. Ver logs da task
aws logs tail /ecs/<service> --follow
```

### Pod EKS não sobe

```bash
# Mesmos comandos do Kubernetes + verificar nodes
kubectl describe pod <nome> -n <ns>
kubectl logs <nome> -n <ns> --previous
aws eks describe-nodegroup --cluster-name <cluster> --nodegroup-name <ng>
kubectl describe node <nome>              # ver se tem recursos disponíveis
```

### Target unhealthy no ALB

```bash
# 1. Ver qual target está unhealthy
aws elbv2 describe-target-health --target-group-arn <arn>

# 2. Se for ECS, ver logs da task
aws logs tail /ecs/<service> --follow --filter-pattern "ERROR"

# 3. Verificar security group — porta do health check liberada?
aws ec2 describe-security-groups --group-ids <sg-id>
```

### Sem acesso / Permission denied

```bash
# 1. Ver quem você é
aws sts get-caller-identity

# 2. Simular a permissão que está falhando
aws iam simulate-principal-policy \
  --policy-source-arn <seu-arn> \
  --action-names <acao>   # ex: ecs:DescribeClusters

# 3. Ver policies anexadas à sua role/user
aws iam list-attached-role-policies --role-name <role>
```

### CloudWatch sem logs aparecendo

```bash
# Verificar se o log group existe
aws logs describe-log-groups --log-group-name-prefix /ecs/<service>

# Verificar log streams (se vazio, container não está enviando)
aws logs describe-log-streams \
  --log-group-name /ecs/<service> \
  --order-by LastEventTime --descending --limit 5

# Verificar se a task tem permissão de escrever no CloudWatch
# A task role precisa ter: logs:CreateLogStream, logs:PutLogEvents
```

---

## 🏴‍☠️ Flags e Truques Úteis

### `--query` — filtrar saída (JMESPath)

```bash
# Pegar só os IDs das instâncias rodando
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text

# Tabela com nome e status dos services ECS
aws ecs describe-services \
  --cluster meu-cluster \
  --services $(aws ecs list-services --cluster meu-cluster --query 'serviceArns' --output text) \
  --query 'services[*].[serviceName,runningCount,desiredCount,status]' \
  --output table
```

### `--output` — formatos de saída

```bash
aws ec2 describe-instances --output json     # padrão — completo
aws ec2 describe-instances --output table    # tabela legível
aws ec2 describe-instances --output text     # texto puro (bom para scripts)
aws ec2 describe-instances --output yaml     # YAML
```

### Variáveis de ambiente úteis

```bash
export AWS_DEFAULT_REGION=us-east-1
export AWS_PROFILE=serasa-dev
export AWS_DEFAULT_OUTPUT=table            # saída em tabela por padrão
```

### Comandos que salvam tempo no dia a dia

```bash
# Ver conta AWS atual
aws sts get-caller-identity --query Account --output text

# Listar tudo que está em ALARM no CloudWatch
aws cloudwatch describe-alarms --state-value ALARM \
  --query 'MetricAlarms[*].[AlarmName,StateReason]' \
  --output table

# Ver logs de erro dos últimos 30 minutos
aws logs filter-log-events \
  --log-group-name /ecs/meu-service \
  --filter-pattern "ERROR" \
  --start-time $(date -d '30 minutes ago' +%s000)

# Forçar redeploy de todos os services de um cluster ECS
for svc in $(aws ecs list-services --cluster meu-cluster --query 'serviceArns[*]' --output text); do
  aws ecs update-service --cluster meu-cluster --service $svc --force-new-deployment
done

# Ver uso de recursos de nodes EKS
kubectl top nodes --sort-by=cpu
kubectl top pods -A --sort-by=memory
```

### Diferença ECS vs EKS — quando cada um aparece

| | ECS | EKS |
|---|---|---|
| Complexidade | Menor | Maior |
| Controle | Menor | Total |
| Quando usar | Apps simples, Fargate | Apps complexas, multi-time |
| Equivalente | Deployment simples | Kubernetes completo |
| Custo | Menor | Maior (control plane pago) |

---

## 📎 Referências

- [AWS CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [AWS ECS Developer Guide](https://docs.aws.amazon.com/ecs/)
- [AWS EKS User Guide](https://docs.aws.amazon.com/eks/)
- [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
