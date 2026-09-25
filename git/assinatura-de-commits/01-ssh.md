# 1. Assinando com SSH

[← Voltar ao índice](README.md)

## 1.1 Verificar se você já tem uma chave

```bash
ls -la ~/.ssh/*.pub
```

- Se aparecer `id_ed25519.pub` (ou `id_rsa.pub`), você já tem uma chave. Pode
  ir direto para o [passo 1.3](#13-configurar-o-git) ou criar uma chave só para
  assinatura (veja o quadro abaixo).
- Se aparecer `No such file or directory`, siga para o passo 1.2.

> 💡 **Mesma chave ou chave separada?** A mesma chave pode servir para
> autenticar (`git push`) e para assinar. É o mais simples. Uma chave só para
> assinatura dá mais controle: se precisar trocá-la, o seu acesso de push não
> é afetado. Para criar uma separada, use `-f ~/.ssh/id_ed25519_signing` no
> passo 1.2 e troque o caminho nos passos seguintes.

## 1.2 Criar uma chave

```bash
ssh-keygen -t ed25519 -C "seu-email@exemplo.com"
```

O que ele pergunta:

1. **`Enter file in which to save the key`** — aperte Enter para aceitar
   `~/.ssh/id_ed25519`.
2. **`Enter passphrase`** — digite uma senha. É **fortemente recomendado**:
   sem ela, quem copiar o arquivo da chave consegue assinar como você.
3. **`Enter same passphrase again`** — repita a senha.

Isso cria dois arquivos:

| Arquivo | O que é | Pode compartilhar? |
| :-- | :-- | :-- |
| `~/.ssh/id_ed25519` | Chave **privada** | ❌ **Nunca** |
| `~/.ssh/id_ed25519.pub` | Chave **pública** | ✅ Sim, é ela que vai para o GitHub |

### Não digitar a senha a cada commit

O `ssh-agent` guarda a chave destravada na memória enquanto a sessão estiver
aberta:

```bash
eval "$(ssh-agent -s)"     # no Ubuntu com interface gráfica, o agente já roda
ssh-add ~/.ssh/id_ed25519  # pede a senha uma vez
ssh-add -l                 # lista as chaves carregadas
```

## 1.3 Configurar o Git

```bash
# Usa SSH em vez de GPG para assinar
git config --global gpg.format ssh

# Qual chave usar (o caminho da chave PÚBLICA, .pub)
git config --global user.signingkey ~/.ssh/id_ed25519.pub

# Assina todos os commits automaticamente, sem precisar de -S
git config --global commit.gpgsign true

# Assina todas as tags anotadas automaticamente
git config --global tag.gpgsign true
```

> ℹ️ A opção se chama `gpg.format` mesmo quando o método é SSH. O nome ficou
> por motivo histórico, já que o Git só assinava com GPG.

Confira:

```bash
git config --global --get-regexp 'gpg|sign'
```

Saída esperada:

```
gpg.format ssh
user.signingkey /home/seu-usuario/.ssh/id_ed25519.pub
commit.gpgsign true
tag.gpgsign true
```

## 1.4 Cadastrar a chave no GitHub

### Pelo site

1. Copie a chave pública:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   Ela começa com `ssh-ed25519 AAAA...` e termina com o seu e-mail.
2. Acesse **GitHub → foto de perfil → Settings → SSH and GPG keys**
   (<https://github.com/settings/keys>).
3. Clique em **New SSH key**.
4. Preencha:
   - **Title:** um nome que identifique a máquina, como `notebook-ubuntu (assinatura)`.
   - **Key type:** **`Signing Key`** ⚠️
   - **Key:** cole a chave pública.
5. Clique em **Add SSH key** e confirme a senha ou o 2FA, se pedir.

> ⚠️ **O erro mais comum de todos:** cadastrar só como *Authentication Key*.
> Nesse caso a chave funciona para `git push`, mas os commits aparecem como
> **Unverified**. Se a mesma chave serve para as duas coisas, ela precisa ser
> cadastrada **duas vezes**, uma de cada tipo. O GitHub permite isso.

### Pelo terminal (GitHub CLI)

```bash
# Dá ao gh a permissão de gerenciar chaves de assinatura (só na primeira vez)
gh auth refresh -h github.com -s admin:ssh_signing_key

# Cadastra a chave como chave de assinatura
gh ssh-key add ~/.ssh/id_ed25519.pub --type signing --title "notebook-ubuntu (assinatura)"

# Confere
gh ssh-key list
```

> ⚠️ A opção `--type` só existe nas versões mais novas do `gh`. Se
> `gh ssh-key add --help` não mostrar `--type`, cadastre a *Signing Key* pelo
> site, ou atualize o `gh` pelo
> [repositório oficial](https://github.com/cli/cli/blob/trunk/docs/install_linux.md).

## 1.5 Permitir a verificação local (opcional, mas recomendado)

O GitHub já verifica as assinaturas do lado dele. Localmente, porém, o Git não
sabe em quais chaves confiar e o `git log --show-signature` mostra este erro:

```
error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification
```

Para resolver, crie um arquivo de "assinantes confiáveis":

```bash
echo "$(git config --global user.email) namespaces=\"git\" $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

Cada linha do arquivo segue o formato:

```
<e-mail> namespaces="git" <tipo-da-chave> <chave> [comentário]
```

Para verificar commits de outras pessoas da equipe, acrescente uma linha para
cada uma. As chaves públicas de qualquer usuário do GitHub ficam em
`https://github.com/<usuario>.keys`.

## 1.6 Testar

```bash
mkdir /tmp/teste-assinatura && cd /tmp/teste-assinatura
git init
git commit --allow-empty -m "Testando assinatura SSH"
git log --show-signature -1
```

Saída esperada:

```
commit 3f2a...
Good "git" signature for seu-email@exemplo.com with ED25519 key SHA256:...
Author: Seu Nome <seu-email@exemplo.com>
...
```

Para testar no GitHub, faça push de um commit para qualquer repositório seu e
veja se o selo **Verified** aparece na lista de commits.

Depois, apague o teste:

```bash
cd ~ && rm -rf /tmp/teste-assinatura
```

---

Próximo: [2. Assinando com GPG →](02-gpg.md) (opcional) ·
[3. Uso no dia a dia →](03-uso-diario.md)
