# 4. Assinando commits que já existem

[← Voltar ao índice](README.md)

Uma assinatura faz parte do commit. Para "assinar depois", é preciso
**recriar** os commits, e isso muda o hash de cada um. Antes de começar,
entenda o que muda em cada caso.

| Situação | Pode reescrever? | Como enviar depois |
| :-- | :-- | :-- |
| Commits **ainda não enviados** (sem push) | ✅ Sem problema | `git push` normal |
| Commits já enviados, **só você** usa a branch | ⚠️ Pode | `git push --force-with-lease` |
| Commits já enviados em branch **compartilhada** (`main` de equipe) | ❌ Evite | Combine antes com a equipe: quem já clonou vai ter histórico divergente |

Antes de tudo, **configure a assinatura** ([SSH](01-ssh.md) ou [GPG](02-gpg.md))
e confirme que `git commit` já assina.

## 4.1 Só o último commit

```bash
git commit --amend --no-edit -S
```

## 4.2 Os últimos N commits

Exemplo com os 3 últimos:

```bash
git rebase HEAD~3 --exec "git commit --amend --no-edit -S"
```

## 4.3 Todos os commits da branch (desde o primeiro)

```bash
git rebase --root --exec "git commit --amend --no-edit -S"
```

### O que o comando faz

- `git rebase --root` refaz todos os commits desde o primeiro.
- `--exec "..."` roda um comando logo depois de reaplicar cada commit.
- `git commit --amend --no-edit -S` recria o commit **assinado**, mantendo a
  mensagem (`--no-edit`) e o autor.

A **data de autoria** é mantida. A **data de commit** (*committer date*) passa
a ser a de agora, e isso é normal.

### Só os commits de uma branch em relação à `main`

```bash
git rebase main --exec "git commit --amend --no-edit -S"
```

## 4.4 Exemplo: reassinando um repositório inteiro

Situação comum: você cria alguns commits, só depois configura a assinatura e
quer que todos apareçam como **Verified** antes do primeiro push.

```bash
# 1. Confira o estado atual (N = sem assinatura)
git log --format='%h %G? %s'

# 2. Reassine todos os commits
git rebase --root --exec "git commit --amend --no-edit -S"

# 3. Confira de novo (agora deve mostrar G)
git log --format='%h %G? %s'
git log --show-signature

# 4. Envie
git push -u origin main
```

> 📝 Arquivos listados em `.git/info/exclude` ou no `.gitignore` não são
> afetados pelo rebase e continuam fora dos commits.

## 4.5 Se algo der errado no meio

```bash
git rebase --abort     # desfaz tudo e volta ao estado de antes
```

Se o rebase já terminou e o resultado ficou errado, o `reflog` guarda o estado
anterior:

```bash
git reflog             # procure a linha anterior ao "rebase (start)"
git reset --hard HEAD@{N}
```

> ⚠️ `git reset --hard` descarta alterações não commitadas. Rode `git status`
> antes para ter certeza de que não há nada pendente.

## 4.6 Depois de reescrever commits já enviados

```bash
git push --force-with-lease
```

Use `--force-with-lease` em vez de `--force`: ele recusa o push se alguém
tiver enviado algo novo nesse meio-tempo, para você não apagar o trabalho de
outra pessoa sem perceber.

Quem já tinha clonado precisa sincronizar:

```bash
git fetch origin
git reset --hard origin/main   # descarta a cópia local antiga da branch
```

---

Próximo: [5. Resolução de problemas →](05-problemas-comuns.md)
