# 0. Pré-requisitos

[← Voltar ao índice](README.md)

## 0.1 Versões das ferramentas

```bash
git --version     # SSH exige 2.34 ou mais nova
ssh -V            # SSH exige OpenSSH 8.1 ou mais nova
gpg --version     # só se for usar GPG
```

Versões com que este guia foi testado (Ubuntu 22.04):

| Ferramenta | Versão | Serve para |
| :-- | :-- | :-- |
| Git | 2.34.1 | ✅ SSH e GPG |
| OpenSSH | 8.9p1 | ✅ SSH |
| GnuPG | 2.2.27 | ✅ GPG |

Se o Git for mais antigo que 2.34 em outra máquina, atualize pelo PPA oficial:

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update && sudo apt install git
```

## 0.2 Nome e e-mail do Git

O GitHub só mostra **Verified** quando o e-mail do commit é um **e-mail
verificado na sua conta**. Confira:

```bash
git config --global user.name
git config --global user.email
```

Para definir:

```bash
git config --global user.name  "Seu Nome"
git config --global user.email "seu-email@exemplo.com"
```

Depois confira em **GitHub → Settings → Emails** se esse e-mail aparece como
verificado (sem o aviso *Unverified*).

### E se eu não quiser expor meu e-mail?

O GitHub oferece um e-mail `noreply` no formato
`12345678+usuario@users.noreply.github.com`. Ele aparece em **Settings →
Emails**, embaixo de *Keep my email addresses private*. Esse endereço conta
como verificado, então pode ser usado em `user.email`.

## 0.3 Global ou por repositório?

- `git config --global ...` grava em `~/.gitconfig` e vale para **todos** os
  repositórios.
- `git config ...` (sem `--global`), rodado dentro de um repositório, grava em
  `.git/config` e vale **só para ele**, com prioridade sobre a global.

Isso ajuda quando você usa e-mails ou chaves diferentes em projetos pessoais
e de trabalho.

Para ver de onde cada configuração vem:

```bash
git config --show-origin --list | grep -E 'user\.|gpg|sign'
```

---

Próximo: [1. Assinando com SSH →](01-ssh.md)
