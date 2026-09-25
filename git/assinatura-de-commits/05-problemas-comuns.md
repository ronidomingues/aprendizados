# 5. Resolução de problemas

[← Voltar ao índice](README.md)

## Diagnóstico geral

Antes de tudo, veja o que o Git está usando:

```bash
git config --show-origin --get-regexp 'user\.|gpg|sign'
```

---

## No GitHub: o commit aparece como **Unverified**

Passe o mouse ou clique no selo: o GitHub diz o motivo. Os mais comuns:

| Mensagem / causa | Solução |
| :-- | :-- |
| *The key whose key-id is … is not yet associated with your account* | A chave não está cadastrada. **SSH:** confira se foi adicionada como **Signing Key**, e não só como Authentication ([1.4](01-ssh.md#14-cadastrar-a-chave-no-github)). **GPG:** cadastre a chave pública ([2.6](02-gpg.md#26-cadastrar-a-chave-no-github)). |
| *The email in this signature doesn't match the committer email* | O `user.email` do commit não é um e-mail verificado da conta, ou (no GPG) é diferente do e-mail da chave. Ajuste `git config user.email` e reassine ([capítulo 4](04-assinando-commits-existentes.md)). |
| *Unverified* em commit **sem** assinatura | O Vigilant mode está ativo e o commit não foi assinado. Confira se `commit.gpgsign` está `true`. |
| A chave GPG expirou | Renove ([2.8](02-gpg.md#renovar-antes-de-expirar)) e recadastre no GitHub. |

---

## Erros de SSH

### `error: Load key "...": invalid format`

`user.signingkey` aponta para a chave **privada** ou para um caminho errado.
Use a **pública**:

```bash
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

### `error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification`

Aparece só na **verificação local**, não impede a assinatura. Configure o
arquivo de assinantes ([1.5](01-ssh.md#15-permitir-a-verificação-local-opcional-mas-recomendado)).

### `No principal matched`

O e-mail do commit não confere com nenhuma linha do `~/.ssh/allowed_signers`.
Confira se o e-mail do arquivo é exatamente igual ao de
`git config user.email`.

### `fatal: ... unknown option: -Y` ou `Couldn't sign message`

A versão do OpenSSH é antiga (precisa ser 8.1 ou mais nova). Rode `ssh -V` e
atualize.

### O Git pede a senha da chave em todo commit

Carregue a chave no agente:

```bash
ssh-add ~/.ssh/id_ed25519
```

### `error: unsupported value for gpg.format: ssh`

O Git é mais antigo que 2.34. Atualize ([0.1](00-pre-requisitos.md#01-versões-das-ferramentas)).

---

## Erros de GPG

### `error: gpg failed to sign the data` / `fatal: failed to write commit object`

É o erro mais comum. Tente, na ordem:

1. **Terminal para a senha:**
   ```bash
   export GPG_TTY=$(tty)
   ```
   Se resolver, coloque no `~/.bashrc` para ficar permanente.
2. **Veja o erro real do GPG:**
   ```bash
   echo "teste" | gpg --clearsign
   ```
3. **Reinicie o agente:**
   ```bash
   gpgconf --kill gpg-agent
   ```
4. **Confira o ID da chave:** o valor de `git config user.signingkey` precisa
   aparecer em `gpg --list-secret-keys --keyid-format=long`.
5. **`gpg.format` ficou como `ssh`** de uma configuração anterior:
   ```bash
   git config --global gpg.format openpgp
   ```

### `fatal: bad config variable 'gpg.format'`

A mensagem completa é parecida com:

```
error: unsupported value for gpg.format: openpg
fatal: bad config variable 'gpg.format' in file '/home/seu-usuario/.gitconfig' at line 17
```

Há um erro de digitação no valor. Os únicos valores aceitos são `openpgp`,
`x509` e `ssh`. Enquanto isso não for corrigido, **todo** comando do Git que
lê assinaturas falha, inclusive o `git log`.

```bash
git config --global gpg.format openpgp
```

### `gpg: signing failed: No secret key`

A chave privada não está nesta máquina. Importe o backup
(`gpg --import chave-privada-BACKUP.asc`) ou crie uma chave nova.

### `gpg: signing failed: Inappropriate ioctl for device`

Falta o `GPG_TTY`. Veja o item 1 acima.

---

## Outros

### O commit foi feito sem assinatura mesmo com tudo configurado

- Confira se não há um `.git/config` local com `commit.gpgsign false`
  sobrepondo a configuração global:
  ```bash
  git config --show-origin commit.gpgsign
  ```
- IDEs e interfaces gráficas às vezes usam outro Git ou ignoram
  `commit.gpgsign`. Procure a opção "Sign commits" nas configurações delas.

### Rebase para assinar para no meio com `doing so would make it empty`

O histórico tem um commit **vazio**, como um `git commit --allow-empty`, e o
`--amend` se recusa a recriá-lo. Acrescente `--allow-empty` ao comando do
`--exec`:

```bash
git rebase --abort      # cancela o rebase que ficou parado
git rebase --root --exec "git commit --amend --no-edit -S --allow-empty"
```

### Quero desligar a assinatura

```bash
git config --global --unset commit.gpgsign
git config --global --unset tag.gpgsign
```

---

## Referências

- [GitHub Docs — Sobre verificação de assinatura de commit](https://docs.github.com/pt/authentication/managing-commit-signature-verification/about-commit-signature-verification)
- [GitHub Docs — Informar o Git sobre sua chave de assinatura](https://docs.github.com/pt/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
- [GitHub Docs — Modo vigilante](https://docs.github.com/pt/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits)
- [Documentação do Git — `git config` (seções `gpg.*`)](https://git-scm.com/docs/git-config#Documentation/git-config.txt-gpgformat)
- [Pro Git — Assinando seu trabalho](https://git-scm.com/book/pt-br/v2/Ferramentas-do-Git-Assinando-o-Seu-Trabalho)

Próximo: [6. Reutilizando a chave GPG em outra máquina →](06-reutilizando-gpg.md)
