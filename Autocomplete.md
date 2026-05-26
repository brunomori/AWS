# 🚀 ZSH + Oh My Zsh + Autocomplete (Estilo iTerm do Mac) no Ubuntu/Linux

Guia para configurar o terminal Linux com:

- ZSH
- Oh My Zsh
- Autocomplete
- Sugestões por histórico
- AWS CLI autocomplete
- Plugins úteis
- Terminal parecido com iTerm do macOS

---

# 📌 1. Instalar dependências

Atualize os pacotes:

```bash
sudo apt update
```

Instale:

```bash
sudo apt install zsh git curl -y
```

Verifique:

```bash
zsh --version
```

---

# 📌 2. Instalar Oh My Zsh

Execute:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Depois confirme o shell padrão:

```bash
echo $SHELL
```

Saída esperada:

```bash
/bin/zsh
```

Se não mudar:

```bash
chsh -s $(which zsh)
```

Depois reinicie sessão ou terminal.

---

# 📌 3. Editar arquivo `.zshrc`

Abra:

```bash
nano ~/.zshrc
```

Esse arquivo controla o comportamento do terminal.

---

# 📌 4. Ativar autocomplete do ZSH

Adicionar:

```bash
autoload -Uz compinit
compinit
```

Isso ativa o TAB completion.

Exemplo:

```bash
aws con + TAB
```

vira:

```bash
aws configure
```

---

# 📌 5. Instalar plugin de sugestões (histórico)

Plugin que sugere comandos já digitados.

Instalar:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

---

# 📌 6. Ativar plugins

No `.zshrc`, localizar:

```bash
plugins=(git)
```

Trocar por:

```bash
plugins=(git aws zsh-autosuggestions)
```

### Explicação

| Plugin | Função |
|---|---|
| git | atalhos Git |
| aws | autocomplete AWS |
| zsh-autosuggestions | sugestões por histórico |

---

# 📌 7. Recarregar terminal

Executar:

```bash
source ~/.zshrc
```

---

# 📌 8. Como funciona agora

### TAB Completion

Digite:

```bash
aws con
```

Pressione:

```bash
TAB
```

Completa:

```bash
aws configure
```

---

### Histórico inteligente

Digite parte do comando:

```bash
aws st
```

Se já usou antes, aparece sugestão cinza.

Aceitar:

```bash
→ (seta direita)
```

ou

```bash
End
```

---

# 📌 9. Autocomplete oficial da AWS CLI

Verifique:

```bash
which aws_completer
```

Normalmente:

```bash
/usr/bin/aws_completer
```

Adicionar no `.zshrc`:

```bash
complete -C '/usr/bin/aws_completer' aws
```

Recarregar:

```bash
source ~/.zshrc
```

Agora comandos AWS ficam muito melhores:

Exemplo:

```bash
aws ec2 des + TAB
```

Completa:

```bash
aws ec2 describe-instances
```

---

# 📌 10. Comandos úteis para testar

## AWS

```bash
aws configure list-profiles
```

```bash
aws sts get-caller-identity
```

```bash
aws s3 ls
```

---

## Git

```bash
git status
```

```bash
git pull
```

---

## Linux

```bash
cd
```

```bash
ls -la
```

```bash
pwd
```

---

# 📌 11. Caso dê erro

## dpkg interrompido

Corrigir:

```bash
sudo dpkg --configure -a
```

Depois:

```bash
sudo apt update
```

---

## zsh não instalado

```bash
sudo apt install zsh
```

---

## plugins não carregando

Recarregar:

```bash
source ~/.zshrc
```

---

# 📌 12. Resultado final

Seu terminal fica parecido com:

✅ autocomplete por TAB  
✅ histórico inteligente  
✅ plugins Git  
✅ autocomplete AWS CLI  
✅ shell mais rápido  
✅ estilo próximo ao iTerm do macOS  
✅ melhor experiência para DevOps / AWS / Kubernetes / SRE

---

# 🎯 Fluxo ideal para quem estuda AWS / K8s / DevOps

Terminal + AWS CLI + kubectl + git + helm + terraform ficam muito mais produtivos.

Exemplo:

```bash
kubectl get po
aws eks update-kubeconfig
helm install
terraform plan
git status
```

---

# 🔥 Setup concluído
Agora seu Linux está com um terminal quase no padrão iTerm + ZSH do macOS.
