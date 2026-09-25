# 7. Reutilizando a chave SSH em outra máquina

[← Voltar ao índice](README.md)

Com SSH existem dois caminhos, e a escolha muda o que você faz no GitHub:

| | **A. Uma chave por máquina** (recomendado) | **B. A mesma chave em todas** |
| :-- | :-- | :-- |
| O que vai para a máquina nova | Nada: gera-se uma chave lá | A chave privada, copiada |
| Cadastro no GitHub | Uma entrada nova por máquina | Nenhuma mudança |
| Se uma máquina for comprometida | Apaga só a chave dela no GitHub | Troca a chave em **todas** |
| Chave privada circulando | Nunca sai da máquina | Precisa ser transportada |

A chave SSH não tem validade nem revogação como a GPG. Por isso, **apagar a
chave no GitHub é o único jeito de "revogá-la"**. Com uma chave por máquina,
isso atinge só o computador perdido.

---

## Caminho A — uma chave por máquina (recomendado)

### A.1 Gerar a chave na máquina nova

```bash
ssh-keygen -t ed25519 -C "seu-email@exemplo.com" -f ~/.ssh/id_ed25519
```

Use uma senha (*passphrase*). Os detalhes estão em
[1.2](01-ssh.md#12-criar-uma-chave).

### A.2 Cadastrar no GitHub

Pelo site, em **Settings → SSH and GPG keys → New SSH key**, cadastre a chave
**duas vezes**: uma como *Authentication Key* (para `git push`) e outra como
*Signing Key* (para assinar). Dê um título com o nome da máquina.

Pelo terminal:

```bash
gh auth refresh -h github.com -s admin:public_key,admin:ssh_signing_key
gh ssh-key add ~/.ssh/id_ed25519.pub --title "nome-da-maquina"                       # autenticação
gh ssh-key add ~/.ssh/id_ed25519.pub --title "nome-da-maquina" --type signing        # assinatura
```

> ⚠️ A opção `--type` só existe nas versões mais novas do `gh`. Se
> `gh ssh-key add --help` não mostrar `--type`, cadastre a *Signing Key* pelo
> site.

### A.3 Configurar o Git

Faça como em [1.3](01-ssh.md#13-configurar-o-git):

```bash
git config --global user.email seu-email@exemplo.com
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

### A.4 Verificação local das chaves das outras máquinas (opcional)

Para o `git log --show-signature` reconhecer commits assinados em **qualquer**
uma das suas máquinas, o `allowed_signers` de cada uma precisa listar todas as
suas chaves. As chaves públicas cadastradas ficam em
`https://github.com/<usuario>.keys`:

```bash
curl -s https://github.com/<usuario>.keys | \
  sed "s/^/seu-email@exemplo.com namespaces=\"git\" /" > ~/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

> O endereço `.keys` lista as chaves de **autenticação**. Por isso, esse
> atalho funciona quando a mesma chave está cadastrada nos dois tipos, como
> no passo A.2.

---

## Caminho B — a mesma chave em todas as máquinas

### B.1 Copiar a chave

A chave são dois arquivos: `~/.ssh/id_ed25519` (privada) e
`~/.ssh/id_ed25519.pub` (pública). Da máquina de origem para a nova:

```bash
scp ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub usuario@maquina-nova:~/.ssh/
```

Ou use um pendrive seu, apagando os arquivos dele depois. **Não** mande a
chave privada por e-mail, chat ou nuvem.

Se a chave tiver senha, o arquivo copiado continua protegido por ela. Para
conferir se tem, rode `ssh-keygen -y -f ~/.ssh/id_ed25519`. Se ele pedir a
senha, a chave está protegida.

### B.2 Acertar as permissões

O SSH **recusa** chaves privadas que outros usuários consigam ler
(`WARNING: UNPROTECTED PRIVATE KEY FILE!`):

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### B.3 Conferir que é a mesma chave

Rode nas duas máquinas e compare os fingerprints:

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

Se só a privada tiver sido copiada, dá para recriar a pública a partir dela:

```bash
ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub
```

### B.4 Carregar no agente e configurar o Git

```bash
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com          # esperado: "Hi <usuario>! You've successfully authenticated..."
```

Depois, faça a mesma configuração do Git do passo A.3. Não é preciso mexer no
GitHub, porque a chave já está cadastrada.

Para a verificação local, copie também o `~/.ssh/allowed_signers` ou recrie-o
como em [1.5](01-ssh.md#15-permitir-a-verificação-local-opcional-mas-recomendado).

---

## 7.1 Particularidades de cada sistema

### Debian, Ubuntu e Kali

```bash
sudo apt install openssh-client git
```

O Git precisa ser 2.34 ou mais novo para assinar com SSH (veja
[0.1](00-pre-requisitos.md#01-versões-das-ferramentas)).

### Windows

O Windows 10 e o 11 já trazem o OpenSSH. A pasta das chaves é
`C:\Users\<voce>\.ssh\`.

1. Ligue o agente SSH (PowerShell **como administrador**):
   ```powershell
   Get-Service ssh-agent | Set-Service -StartupType Automatic
   Start-Service ssh-agent
   ssh-add $env:USERPROFILE\.ssh\id_ed25519
   ```
2. No **caminho B**, restrinja a permissão da chave copiada:
   ```powershell
   icacls $env:USERPROFILE\.ssh\id_ed25519 /inheritance:r /grant:r "$($env:USERNAME):F"
   ```
3. Se o Git pedir a senha da chave em todo commit, faça-o usar o `ssh-keygen`
   do Windows, que conversa com o agente do sistema:
   ```powershell
   git config --global gpg.ssh.program "C:/Windows/System32/OpenSSH/ssh-keygen.exe"
   ```

### macOS

Para guardar a senha no Keychain:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

E, em `~/.ssh/config`:

```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

### Android (Termux)

```bash
pkg install openssh git
```

As chaves ficam em `~/.ssh/`, dentro do Termux. Para carregar a chave no
agente a cada sessão:

```bash
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519
```

## 7.2 Testar

```bash
cd /tmp && git init teste-ssh && cd teste-ssh
git commit --allow-empty -m "Testando a chave SSH nesta máquina"
git log --show-signature -1       # esperado: Good "git" signature for ...
cd .. && rm -rf teste-ssh
```

Para conferir no GitHub, faça push de um commit e veja o selo **Verified**.

## 7.3 Tirar a chave de uma máquina

| Caminho | O que fazer |
| :-- | :-- |
| **A** (uma por máquina) | Apague a chave da máquina em **GitHub → Settings → SSH and GPG keys**, nas duas entradas (autenticação e assinatura). Depois apague os arquivos locais. |
| **B** (a mesma em todas) | Apague só os arquivos locais (`rm ~/.ssh/id_ed25519*`). **Não** apague no GitHub, senão as outras máquinas param de funcionar. |

Se a máquina foi **comprometida** no caminho B, a chave vazou para todas:
gere uma chave nova, cadastre no GitHub, apague a antiga lá e troque em todas
as máquinas. Essa troca geral é justamente o que o caminho A evita.

---

Próximo: [8. Relato: como eu configurei →](08-relato.md)
