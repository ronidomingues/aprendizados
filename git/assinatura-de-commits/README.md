# Configurando assinaturas de commits no GitHub

Guia para assinar commits e tags no Git e fazer o GitHub mostrar o selo
**Verified** ao lado de cada um.

## Por que assinar?

O Git aceita qualquer nome e e-mail em `git config user.name` e `user.email`.
Qualquer pessoa pode criar um commit dizendo que foi você. Uma assinatura
criptográfica prova que o commit saiu de alguém que tem a sua chave privada.
O GitHub confere essa assinatura e mostra:

| Selo | Significado |
| :-- | :-- |
| **Verified** | A assinatura é válida e a chave está cadastrada na sua conta. |
| **Unverified** | Há uma assinatura, mas ela não pôde ser validada (chave desconhecida, e-mail não confere etc.). |
| *(nenhum)* | O commit não foi assinado. Com o [Vigilant mode](03-uso-diario.md#vigilant-mode) ativo, ele aparece como **Unverified**. |

## Qual método escolher?

| | SSH | GPG |
| :-- | :-- | :-- |
| Dificuldade | ⭐ Fácil | ⭐⭐⭐ Trabalhoso |
| Reaproveita a chave que você já usa para `git push` | ✅ Sim | ❌ Não |
| Versão mínima do Git | 2.34 | Qualquer uma |
| Expiração e revogação de chave | ❌ Não tem | ✅ Tem |
| Padrão tradicional (kernel, distros Linux) | ❌ | ✅ |

**Recomendação:** use **SSH**, a não ser que um projeto exija GPG.

## Conteúdo

1. [Pré-requisitos](00-pre-requisitos.md)
2. [Assinando com SSH](01-ssh.md) ← **comece por aqui**
3. [Assinando com GPG](02-gpg.md)
4. [Usando e verificando assinaturas no dia a dia](03-uso-diario.md)
5. [Assinando commits que já existem](04-assinando-commits-existentes.md)
6. [Resolução de problemas](05-problemas-comuns.md)
7. [Relato: como eu configurei](06-relato.md) ← o que eu fiz, errei e aprendi

## Resumo rápido (SSH)

```bash
# 1. Chave (pule se já tiver ~/.ssh/id_ed25519.pub)
ssh-keygen -t ed25519 -C "seu-email@exemplo.com"

# 2. Git
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# 3. GitHub → Settings → SSH and GPG keys → New SSH key → Key type: "Signing Key"
cat ~/.ssh/id_ed25519.pub

# 4. Teste
git commit --allow-empty -m "Testando assinatura"
git log --show-signature -1
```
