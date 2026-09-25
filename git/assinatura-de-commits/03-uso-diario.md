# 3. Usando e verificando assinaturas no dia a dia

[← Voltar ao índice](README.md)

## 3.1 Assinando

Com `commit.gpgsign true`, você **não precisa fazer nada**: todo `git commit`
sai assinado.

Sem essa opção, ou para forçar em um caso pontual:

```bash
git commit -S -m ":memo: docs(readme): Atualizando o README"
```

Para **não** assinar um commit específico:

```bash
git commit --no-gpg-sign -m "..."
```

### Tags

```bash
git tag -s v1.0.0 -m "Versão 1.0.0"   # -s cria uma tag anotada e assinada
git tag -v v1.0.0                      # verifica a assinatura da tag
git push origin v1.0.0
```

> Com `tag.gpgsign true`, `git tag -a` já assina. Tags leves (`git tag v1.0.0`,
> sem `-a`, `-s` ou `-m`) **não podem** ser assinadas.

## 3.2 Verificando

### O último commit

```bash
git verify-commit HEAD
```

### Com detalhes no log

```bash
git log --show-signature -5
```

### Em formato de tabela

```bash
git log --format='%h %G? %<(20,trunc)%GS %s' -20
```

O código `%G?` significa:

| Código | Significado |
| :-: | :-- |
| `G` | Assinatura **válida** ✅ |
| `B` | Assinatura **inválida** (commit adulterado) ❌ |
| `U` | Válida, mas de chave sem confiança definida |
| `X` / `Y` | Válida, mas a assinatura ou a chave expirou |
| `R` | Chave revogada |
| `E` | Não foi possível verificar (chave ausente, `allowed_signers` não configurado etc.) |
| `N` | **Sem assinatura** |

Um atalho para usar sempre:

```bash
git config --global alias.lg-sig "log --format='%C(yellow)%h%Creset %G? %C(cyan)%an%Creset %s'"
git lg-sig -10
```

### No GitHub

- Na lista de commits (`/commits`), cada commit mostra o selo à direita.
  Clicar nele exibe a chave usada.
- Para uma checagem mais rigorosa, ative o **Vigilant mode** (veja a seguir).

### Vigilant mode

O Git aceita qualquer e-mail em `user.email`. Sem o Vigilant mode, um commit
falso com o seu e-mail fica **sem selo**, igual a um commit seu que não foi
assinado, e ninguém percebe a diferença. Com ele ativo, o GitHub marca como
**Unverified** todo commit com o seu e-mail que não tenha a sua assinatura.

| Commit com o seu e-mail | Sem Vigilant mode | Com Vigilant mode |
| :-- | :-- | :-- |
| Assinado com a sua chave | ✅ Verified | ✅ Verified |
| Sem assinatura | *(nenhum selo)* | ⚠️ **Unverified** |
| Assinado por você, mas com outra pessoa como autora | ✅ Verified | 🟡 **Partially verified** |

**Como ativar:** **GitHub → Settings → SSH and GPG keys**
(<https://github.com/settings/keys>) → marque **Flag unsigned commits as
unverified**.

**O que muda depois de ativar:**

- **Commits antigos:** todos os seus commits antigos **sem assinatura**, em
  qualquer repositório, passam a aparecer como **Unverified**. O conteúdo não
  muda, só o selo.
- **Outras máquinas:** um commit feito num computador ou numa ferramenta sem
  assinatura configurada também aparece como Unverified. Configure a
  assinatura em cada máquina ([3.3](#33-várias-máquinas)).
- **Commits feitos pelo site do GitHub** (editar arquivo, merge de PR)
  continuam **Verified**, porque o GitHub assina com a chave dele.
- **Não bloqueia nada:** o Vigilant mode só muda o selo exibido. Para o GitHub
  recusar o push de commits sem assinatura, use a regra **Require signed
  commits** ([3.5](#35-exigir-assinatura-em-um-repositório)).

## 3.3 Várias máquinas

Cada computador precisa da própria configuração:

- **SSH:** gere uma chave por máquina e cadastre **cada uma** como *Signing
  Key* no GitHub. É mais seguro que copiar a chave privada entre máquinas.
- **GPG:** importe o backup da chave (`gpg --import`) ou crie uma subchave por
  máquina.

## 3.4 Commits feitos pela interface do GitHub

Commits criados pelo site do GitHub (editar arquivo, *merge* de Pull Request,
*squash*) são assinados pela chave do próprio GitHub e aparecem como
**Verified**, sem configuração nenhuma.

## 3.5 Exigir assinatura em um repositório

Em **Repositório → Settings → Branches (ou Rules) → Add rule**, marque
**Require signed commits**. A partir daí, o GitHub recusa push de commits não
assinados na branch protegida.

---

Próximo: [4. Assinando commits que já existem →](04-assinando-commits-existentes.md)
