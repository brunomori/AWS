# AWS Cost Audit --- Pós-Lab / Anti-Custo

## Contexto

Este repositório/documentação foi criado após uma auditoria completa em
uma conta AWS usada para estudos e labs.

Objetivo principal: **evitar cobranças inesperadas** (principalmente NAT
Gateway, EKS, EC2 e recursos órfãos).

Durante a auditoria, foram identificados custos anteriores em:

-   NAT Gateway
-   EKS (Elastic Kubernetes Service)
-   Recursos de VPC associados
-   CloudWatch Logs

Esses recursos foram auditados e removidos.

------------------------------------------------------------------------

## O que aconteceu

A conta apresentou cobrança em uma fatura anterior (\~USD 3.49),
principalmente por:

### NAT Gateway

Custos por hora mesmo sem tráfego relevante.

### EKS

Cluster de estudo gerando custo.

### VPC / Infra associada

Infra de rede residual do lab.

Depois da limpeza, a conta ficou apenas com infraestrutura
padrão/grátis.

------------------------------------------------------------------------

## Recursos auditados

Verificados:

-   EC2 Instances
-   EBS Volumes
-   Snapshots
-   Elastic IPs
-   NAT Gateways
-   Load Balancers
-   RDS
-   EKS
-   ECS
-   Lambda
-   ECR
-   S3
-   OpenSearch
-   ElastiCache
-   Route53
-   CloudFormation
-   CloudWatch Logs
-   Network Interfaces
-   VPCs
-   Subnets
-   Route Tables
-   Security Groups
-   Internet Gateways
-   SNS Topic (Billing Alarm)

------------------------------------------------------------------------

## Script de Auditoria

Arquivo: `aws-cost-audit.sh`

``` bash
#!/bin/bash

REGION="us-east-1"

echo "======================================="
echo " AWS COST AUDIT"
echo " Conta atual:"
aws sts get-caller-identity
echo "======================================="

echo -e "\n[EC2 INSTANCES]"
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
  --output table

echo -e "\n[EBS VOLUMES]"
aws ec2 describe-volumes \
  --query "Volumes[*].[VolumeId,State,Size]" \
  --output table

echo -e "\n[SNAPSHOTS]"
aws ec2 describe-snapshots \
  --owner-ids self \
  --query "Snapshots[*].[SnapshotId,VolumeSize]" \
  --output table

echo -e "\n[ELASTIC IP]"
aws ec2 describe-addresses \
  --query "Addresses[*].[PublicIp,AllocationId]" \
  --output table

echo -e "\n[NAT GATEWAYS]"
aws ec2 describe-nat-gateways \
  --query "NatGateways[*].[NatGatewayId,State]" \
  --output table

echo -e "\n[LOAD BALANCERS]"
aws elbv2 describe-load-balancers \
  --query "LoadBalancers[*].[LoadBalancerArn,State.Code]" \
  --output table

echo -e "\n[RDS]"
aws rds describe-db-instances \
  --query "DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]" \
  --output table

echo -e "\n[EKS]"
aws eks list-clusters

echo -e "\n[ECS]"
aws ecs list-clusters

echo -e "\n[LAMBDA]"
aws lambda list-functions \
  --query "Functions[*].[FunctionName,Runtime]" \
  --output table

echo -e "\n[ECR]"
aws ecr describe-repositories \
  --query "repositories[*].[repositoryName]" \
  --output table

echo -e "\n[S3 BUCKETS]"
aws s3 ls

echo -e "\n[OPENSEARCH]"
aws opensearch list-domain-names

echo -e "\n[ELASTICACHE]"
aws elasticache describe-cache-clusters \
  --query "CacheClusters[*].[CacheClusterId,Engine]" \
  --output table

echo -e "\n[ROUTE53]"
aws route53 list-hosted-zones \
  --query "HostedZones[*].[Name,Id]" \
  --output table

echo -e "\n[CLOUDFORMATION]"
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

echo -e "\n[CLOUDWATCH LOG GROUPS]"
aws logs describe-log-groups \
  --query "logGroups[*].[logGroupName,storedBytes]" \
  --output table

echo -e "\n[NETWORK INTERFACES]"
aws ec2 describe-network-interfaces \
  --query "NetworkInterfaces[*].[NetworkInterfaceId,Status]" \
  --output table

echo -e "\n[INTERNET GATEWAYS]"
aws ec2 describe-internet-gateways \
  --query "InternetGateways[*].[InternetGatewayId]" \
  --output table

echo -e "\n[SECURITY GROUPS]"
aws ec2 describe-security-groups \
  --query "SecurityGroups[*].[GroupId,GroupName]" \
  --output table

echo -e "\n[VPCS]"
aws ec2 describe-vpcs \
  --query "Vpcs[*].[VpcId,IsDefault,State]" \
  --output table

echo -e "\n[SUBNETS]"
aws ec2 describe-subnets \
  --query "Subnets[*].[SubnetId,VpcId]" \
  --output table

echo -e "\n[ROUTE TABLES]"
aws ec2 describe-route-tables \
  --query "RouteTables[*].[RouteTableId,VpcId]" \
  --output table

echo -e "\n[SNS TOPICS]"
aws sns list-topics --region $REGION

echo "======================================="
echo " AUDIT FINISHED"
echo "======================================="
```

------------------------------------------------------------------------

## Como usar

### Dar permissão

``` bash
chmod +x aws-cost-audit.sh
```

### Rodar

``` bash
./aws-cost-audit.sh
```

------------------------------------------------------------------------

## Resultado esperado (conta limpa)

Normalmente deve aparecer vazio para:

-   EC2
-   NAT Gateway
-   RDS
-   EKS
-   ECS
-   Lambda
-   ECR
-   S3
-   Load Balancers
-   Snapshots
-   Elastic IP
-   Network Interfaces

------------------------------------------------------------------------

## Recursos que podem aparecer e ainda serem normais

Esses podem existir sem custo relevante:

-   VPC default
-   Internet Gateway default
-   Security Group default
-   Route Table default
-   SNS Topic
-   IAM User

------------------------------------------------------------------------

## Boas práticas

### Antes do lab

``` bash
./aws-cost-audit.sh
```

### Depois do lab

``` bash
./aws-cost-audit.sh
```

### Revisão semanal

Verificar AWS Billing Dashboard.

------------------------------------------------------------------------

## Segurança

Evitar usar root para labs.

Preferir: - IAM user limitado - Billing Alarm - SNS Email confirmado -
CLI profile dedicado

------------------------------------------------------------------------

## Aprendizado principal

Os maiores vilões de custo inesperado em labs AWS normalmente são:

-   NAT Gateway
-   EKS
-   Load Balancer
-   RDS
-   EC2 esquecida
-   Elastic IP órfão
-   EBS Volume esquecido
-   CloudWatch Logs crescendo

------------------------------------------------------------------------

## Status final da auditoria desta conta

Conta ficou:

-   limpa
-   sem recursos clássicos de custo recorrente
-   pronta para estudos futuros
-   com baseline reutilizável
