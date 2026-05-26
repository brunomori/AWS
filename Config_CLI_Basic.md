# ☁️ AWS CLI - Setup Inicial (Profiles, Auth, Config e ZSH)

Guia inicial para configurar AWS CLI no Linux/Ubuntu, entender profiles, autenticação e fluxo básico usado em times DevOps / SRE / Cloud.

---

# 📌 1. Instalar AWS CLI

Atualizar pacotes:

```bash
sudo apt update
```

Instalar dependências:

```bash
sudo apt install unzip curl -y
```

Baixar AWS CLI:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

Extrair:

```bash
unzip awscliv2.zip
```

Instalar:

```bash
sudo ./aws/install
```

Validar:

```bash
aws --version
```

Saída esperada:

```bash
aws-cli/2.x.x
```

---

# 📌 2. Configurar primeiro profile

Executar:

```bash
aws configure
```

Vai pedir:

```bash
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

Exemplo:

```bash
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: ********
Default region name [None]: us-east-1
Default output format [None]: json
```

---

# 📌 3. Estrutura de arquivos AWS CLI

AWS salva configuração em:

```bash
~/.aws/
```

Arquivos principais:

```bash
~/.aws/credentials
~/.aws/config
```

---

## credentials

Guarda credenciais.

```ini
[default]
aws_access_key_id=AKIA...
aws_secret_access_key=xxxxx
```

---

## config

Guarda região, output e profiles.

```ini
[default]
region=us-east-1
output=json
```

---

# 📌 4. Criar vários profiles

Exemplo:

```bash
aws configure --profile dev
aws configure --profile prod
aws configure --profile lab
```

Agora terá múltiplos ambientes.

---

# 📌 5. Ver profiles existentes

```bash
aws configure list-profiles
```

Exemplo:

```bash
default
dev
prod
lab
```

---

# 📌 6. Ver qual profile está ativo

```bash
echo $AWS_PROFILE
```

Se vier vazio:

```bash
(default)
```

---

# 📌 7. Trocar profile manualmente

Temporário:

```bash
export AWS_PROFILE=dev
```

Validar:

```bash
echo $AWS_PROFILE
```

---

Voltar pro default:

```bash
unset AWS_PROFILE
```

ou

```bash
export AWS_PROFILE=default
```

---

# 📌 8. Rodar comando com outro profile sem trocar global

Muito usado.

```bash
aws s3 ls --profile dev
```

```bash
aws eks list-clusters --profile prod
```

```bash
aws lambda list-functions --profile lab
```

Isso afeta só o comando.

---

# 📌 9. Testar autenticação AWS

Comando mais importante:

```bash
aws sts get-caller-identity
```

Se estiver autenticado:

```json
{
  "UserId": "...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/..."
}
```

Esse comando confirma:

- credenciais válidas
- profile funcionando
- conta AWS correta
- role/usuário atual

---

# 📌 10. Comandos básicos de verificação

## S3

```bash
aws s3 ls
```

Lista buckets.

---

## EC2

```bash
aws ec2 describe-regions
```

Lista regiões AWS.

---

## EKS

```bash
aws eks list-clusters
```

Lista clusters EKS.

---

## Lambda

```bash
aws lambda list-functions
```

Lista funções.

---

# 📌 11. Entender se a key expira

## IAM User (key fixa)

Se no credentials tiver:

```ini
aws_access_key_id
aws_secret_access_key
```

Normalmente não expira automaticamente.

---

## Token temporário

Se aparecer:

```ini
aws_session_token
```

É temporário.

Pode expirar:
- 1h
- 8h
- 12h
- conforme política da empresa

Erro comum:

```bash
ExpiredToken
```

---

# 📌 12. Ver arquivos AWS

Mostrar config:

```bash
cat ~/.aws/config
```

Mostrar credentials:

```bash
cat ~/.aws/credentials
```

Nunca subir isso no GitHub.

Nunca compartilhar Access Key.

---

# 📌 13. Criar atalho no ZSH (set_aws_profile)

Abrir:

```bash
nano ~/.zshrc
```

Adicionar:

```bash
set_aws_profile() {
  if [[ -z "$1" ]]; then
    echo "Uso: set_aws_profile <nome-do-profile>"
    echo ""
    echo "Perfis disponíveis:"
    aws configure list-profiles
    return 1
  fi

  export AWS_PROFILE=$1
  echo "AWS Profile: $AWS_PROFILE"
  aws sts get-caller-identity
}
```

Recarregar:

```bash
source ~/.zshrc
```

---

## Uso

Trocar:

```bash
set_aws_profile dev
```

```bash
set_aws_profile prod
```

```bash
set_aws_profile lab
```

---

Sem parâmetro:

```bash
set_aws_profile
```

Lista profiles.

---

# 📌 14. Alias curto (opcional)

Adicionar no `.zshrc`:

```bash
alias sap='set_aws_profile'
```

Uso:

```bash
sap dev
sap prod
sap lab
```

---

# 📌 15. Erros comuns

## ExpiredToken

Token expirou.

Refazer login ou renovar credencial.

---

## Unable to locate credentials

Sem credenciais configuradas.

Rodar:

```bash
aws configure
```

---

## AccessDenied

Usuário/Role sem permissão IAM.

---

## InvalidClientTokenId

Key inválida ou desativada.

---

# 📌 16. Fluxo ideal DevOps / SRE

Ver perfis:

```bash
aws configure list-profiles
```

Trocar:

```bash
set_aws_profile dev
```

Confirmar:

```bash
aws sts get-caller-identity
```

Testar:

```bash
aws s3 ls
aws eks list-clusters
aws ec2 describe-regions
```

---

# 🎯 Conclusão

Esse setup cobre o básico que praticamente todo ambiente AWS usa:

✅ AWS CLI  
✅ Profiles  
✅ Auth  
✅ Config  
✅ Credentials  
✅ STS Validation  
✅ ZSH Alias  
✅ Multi-account workflow  
✅ Base para EKS / Lambda / S3 / DevOps / SRE
