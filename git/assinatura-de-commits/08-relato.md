# 6. Relato: como eu configurei

[← Voltar ao índice](README.md)

O guia dos capítulos anteriores saiu daqui: o que eu fiz, na ordem em que fiz,
com os erros incluídos. Tudo aconteceu em **25/09/2026**, num Ubuntu 22.04.

## 6.1 O ponto de partida

Sem assinatura, um commit meu no GitHub não tinha selo nenhum. Qualquer
pessoa que configurasse o meu e-mail no Git conseguiria criar commits
idênticos aos meus.

## 6.2 Primeira tentativa: SSH

Comecei pelo SSH, porque é o caminho mais curto ([capítulo 1](01-ssh.md)):
uma chave `ed25519`, quatro linhas de `git config` e o cadastro no GitHub como
**Signing Key**. Funcionou de primeira, e os commits passaram a sair com `G`
no `git log --format='%G?'`.

## 6.3 A troca para GPG

No mesmo dia, migrei para GPG ([capítulo 2](02-gpg.md)). Diferente da chave
SSH, a GPG tem data de validade e certificado de revogação. Criei uma chave
RSA 4096 que vale até maio de 2027, cadastrei no GitHub e troquei o
`gpg.format` e o `user.signingkey`.

### O erro: `openpg`

Na troca, digitei `gpg.format = openpg`, faltando o último "p". O Git não
avisa na hora da configuração. O erro só aparece no próximo comando que lê
assinaturas, e trava até o `git log`:

```
error: unsupported value for gpg.format: openpg
fatal: bad config variable 'gpg.format' in file '/home/seu-usuario/.gitconfig' at line 17
```

A correção foi uma linha: `git config --global gpg.format openpgp`. Registrei
o caso em [Resolução de problemas](05-problemas-comuns.md#fatal-bad-config-variable-gpgformat).

**Lição:** depois de mexer em configuração de assinatura, rode
`git log --show-signature -1` na hora, antes de seguir.

## 6.4 Reassinando commits que já estavam no GitHub

Os commits de um dos repositórios tinham sido assinados com SSH e já estavam
na `main` do GitHub. Para trocar pela assinatura GPG, precisei reescrever o
histórico ([capítulo 4](04-assinando-commits-existentes.md)):

```bash
git branch backup-antes-gpg                      # rede de segurança
git rebase --root --exec "git commit --amend --no-edit -S"
git diff --stat backup-antes-gpg HEAD            # vazio = conteúdo intacto
git push --force-with-lease=main:<hash-antigo> origin main
```

Três cuidados fizeram diferença:

- **Branch de backup antes do rebase.** Se algo desse errado, bastava voltar
  para ela.
- **`git diff` entre o backup e o resultado.** Comprova que só as assinaturas
  mudaram e que o conteúdo ficou igual.
- **`--force-with-lease` preso ao hash antigo.** O push só substitui a `main`
  se ninguém tiver enviado nada nesse meio-tempo.

A API do GitHub confirmou o resultado:

```bash
gh api "repos/<dono>/<repo>/commits" --jq '.[] | "\(.sha[0:7]) \(.commit.verification.verified) \(.commit.verification.reason)"'
```

```
3016991 true valid
c3b7a55 true valid
```

### Outro tropeço: commits vazios

Ao testar o rebase num repositório de mentira, com commits criados por
`git commit --allow-empty`, o `--amend` se recusou a continuar
(`doing so would make it empty`). Em repositórios reais isso quase não
acontece, mas a solução ficou documentada: acrescentar `--allow-empty` ao
`--exec`.

## 6.5 Vigilant mode

Por fim, ativei o **Vigilant mode** ([3.2](03-uso-diario.md#vigilant-mode)).
Agora qualquer commit com o meu e-mail que não tenha a minha assinatura
aparece como **Unverified**. Antes, ele passaria sem selo e ninguém notaria.

## 6.6 Como ficou

- Todo commit e toda tag saem assinados automaticamente com GPG.
- A chave pública está no GitHub. Qualquer pessoa pode conferir as minhas
  chaves em <https://github.com/ronidomingues.gpg>.
- O Vigilant mode está ativo.
- Ficaram anotadas para mim duas tarefas: guardar o backup da chave privada
  fora do computador e renovar a chave antes de ela expirar
  ([2.8](02-gpg.md#28-cuidando-da-chave-gpg)).

## 6.7 O que eu levo disso

1. **SSH é o jeito mais rápido de começar.** GPG dá mais controle, mas pede
   mais cuidado.
2. **Teste logo depois de configurar.** Um erro de digitação passou em
   silêncio até o comando seguinte.
3. **Reescrever histórico público é seguro com rede de proteção:** backup,
   `diff` e `--force-with-lease`.
4. **Assinar sem o Vigilant mode protege só pela metade.** Um commit falso
   continua sem aparecer como suspeito.

---

[← Voltar ao índice](README.md)
