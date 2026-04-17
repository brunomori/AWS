# ⚡ AWS Lambda - Guia Prático (Serverless)

Guia completo para criação, configuração e execução de funções **AWS Lambda**.

---

# 📌 1. Conceito

O **AWS Lambda** é um serviço **serverless** que permite executar código sem gerenciar servidores.

Você apenas:

* 📦 Sobe o código
* ⚡ Define o gatilho (trigger)
* 🚀 A AWS executa automaticamente

👉 Você paga apenas pelo tempo de execução.

---

# 🧠 2. Casos de uso

* APIs (com API Gateway)
* Processamento de arquivos (S3)
* Automação (CloudWatch Events)
* Integração com serviços AWS (SQS, DynamoDB, etc.)

---

# 🧱 3. Componentes principais

* **Function** → Código executado
* **Trigger** → Evento que dispara a execução
* **IAM Role** → Permissões da função
* **Logs (CloudWatch)** → Monitoramento

---

# 🔐 4. Criar IAM Role para Lambda

## 📄 Trust Policy

```json id="c8u9t2"
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 🚀 Criar Role via CLI

```bash id="k3x9wd"
aws iam create-role \
  --role-name lambda-execution-role \
  --assume-role-policy-document file://trust-policy.json
```

---

## 🔗 Attach policy básica

```bash id="t6v1m8"
aws iam attach-role-policy \
  --role-name lambda-execution-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

👉 Essa policy permite enviar logs para o CloudWatch.

---

# ⚡ 5. Criar função Lambda via CLI

## 📄 Exemplo código (Python)

```python id="z9p2q1"
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Hello from Lambda 🚀"
    }
```

---

## 📦 Zipar código

```bash id="j8n2c0"
zip function.zip lambda_function.py
```

---

## 🚀 Criar função

```bash id="m4d7q9"
aws lambda create-function \
  --function-name minha-lambda \
  --runtime python3.9 \
  --role arn:aws:iam::SEU_ACCOUNT_ID:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

---

# ▶️ 6. Executar (invocar) função

```bash id="f2w6z3"
aws lambda invoke \
  --function-name minha-lambda \
  response.json
```

---

# 📊 7. Resultado esperado

Arquivo `response.json`:

```json id="p1s8y4"
{
  "statusCode": 200,
  "body": "Hello from Lambda 🚀"
}
```

---

# 🌐 8. Expor como API (API Gateway)

Fluxo:

1. Criar API Gateway
2. Integrar com Lambda
3. Criar endpoint HTTP

👉 Resultado:

```text id="q7x2a6"
https://xxxxx.execute-api.us-east-1.amazonaws.com/prod
```

---

# 🧪 9. Teste via curl

```bash id="d4c9b1"
curl https://SEU_ENDPOINT
```

---

# ⚠️ 10. Erros comuns

* ❌ Role sem permissão
* ❌ Handler incorreto
* ❌ Runtime errado
* ❌ Zip mal estruturado
* ❌ Timeout muito baixo

---

# 🧠 11. Boas práticas

* 🔐 Use IAM mínimo necessário
* ⚡ Configure timeout corretamente
* 📦 Mantenha funções pequenas
* 🔁 Use versionamento (Lambda versions)
* 📊 Monitore via CloudWatch

---

# 📈 12. Observabilidade

Ver logs:

```bash id="u2n6x5"
aws logs describe-log-groups
```

👉 Logs ficam em:

```text id="r5y8k2"
/aws/lambda/minha-lambda
```

---

# 🎯 13. Conclusão

O AWS Lambda permite:

* ⚡ Executar código sem servidor
* 💰 Pagar apenas pelo uso
* 🔗 Integrar facilmente com AWS
* 🚀 Escalar automaticamente

👉 Base essencial para arquiteturas modernas (serverless).
