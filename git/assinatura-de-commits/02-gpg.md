# 2. Assinando com GPG

[← Voltar ao índice](README.md)

> Só siga este capítulo se preferir GPG ou se um projeto exigir. Se já
> configurou SSH, **não precisa** disto. Os dois métodos não se somam: o Git
> usa o que estiver em `gpg.format`.

## 2.1 Instalar

No Ubuntu o GnuPG já vem instalado. Se faltar:

```bash
sudo apt install gnupg
```

## 2.2 Ver se já existe uma chave

```bash
gpg --list-secret-keys --keyid-format=long
```

Se não aparecer nada, siga para o passo 2.3.

## 2.3 Criar uma chave

```bash
gpg --full-generate-key
```

Respostas recomendadas:

| Pergunta | Resposta |
| :-- | :-- |
| *Please select what kind of key you want* | `9` — **ECC (sign and encrypt)**. Em versões antigas, use `1` (RSA and RSA). |
| *Please select which elliptic curve you want* | `1` — **Curve 25519** |
| *What keysize do you want?* (só com RSA) | `4096` |
| *Key is valid for?* | `2y` (2 anos). Dá para renovar depois; veja o passo 2.8. |
| *Real name* | Seu nome, igual ao `user.name` do Git |
| *Email address* | **O mesmo e-mail** de `user.email` e verificado no GitHub ⚠️ |
| *Comment* | Deixe vazio ou escreva algo como `GitHub` |
| *Passphrase* | Uma senha forte |

> ⚠️ Com GPG, o GitHub exige que o **e-mail da chave** seja igual ao **e-mail
> do commit** e que esse e-mail esteja verificado na sua conta.

## 2.4 Descobrir o ID da chave

```bash
gpg --list-secret-keys --keyid-format=long
```

A saída se parece com:

```
sec   ed25519/3AA5C34371567BD2 2026-09-25 [SC] [expires: 2028-09-25]
      1234567890ABCDEF1234567890ABCDEF3AA5C343
uid                 [ultimate] Seu Nome <seu-email@exemplo.com>
ssb   cv25519/42B317FD4BA89E7A 2026-09-25 [E] [expires: 2028-09-25]
```

O **ID** é o trecho depois da barra na linha `sec`: `3AA5C34371567BD2`.

## 2.5 Configurar o Git

```bash
git config --global gpg.format openpgp          # é o padrão; necessário se antes usava SSH
git config --global user.signingkey 3AA5C34371567BD2
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

### Fazer o GPG conseguir pedir a senha

Sem isto, é comum aparecer `error: gpg failed to sign the data`. Adicione ao
`~/.bashrc`:

```bash
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
source ~/.bashrc
```

### Pedir a senha com menos frequência (opcional)

Crie ou edite `~/.gnupg/gpg-agent.conf`:

```
default-cache-ttl 3600
max-cache-ttl 86400
```

Com isso, a senha fica guardada por 1 hora desde o último uso, e por no máximo
24 horas. Para aplicar:

```bash
gpgconf --kill gpg-agent
```

## 2.6 Cadastrar a chave no GitHub

### Pelo site

1. Exporte a chave pública:
   ```bash
   gpg --armor --export 3AA5C34371567BD2
   ```
2. Copie **tudo**, inclusive as linhas
   `-----BEGIN PGP PUBLIC KEY BLOCK-----` e `-----END PGP PUBLIC KEY BLOCK-----`.
3. Acesse **GitHub → Settings → SSH and GPG keys → New GPG key**.
4. Dê um título, cole a chave e clique em **Add GPG key**.

### Pelo terminal (GitHub CLI)

```bash
gh auth refresh -h github.com -s write:gpg_key
gpg --armor --export 3AA5C34371567BD2 > /tmp/minha-chave.asc
gh gpg-key add /tmp/minha-chave.asc --title "notebook-ubuntu"
rm /tmp/minha-chave.asc
```

## 2.7 Testar

```bash
mkdir /tmp/teste-gpg && cd /tmp/teste-gpg && git init
git commit --allow-empty -m "Testando assinatura GPG"
git log --show-signature -1
cd ~ && rm -rf /tmp/teste-gpg
```

Saída esperada:

```
gpg: Signature made ...
gpg: Good signature from "Seu Nome <seu-email@exemplo.com>" [ultimate]
```

## 2.8 Cuidando da chave GPG

### Backup (faça!)

Se perder a chave privada, não dá para renovar nem revogar. Guarde um backup
**fora do computador**, em um pendrive ou num gerenciador de senhas:

```bash
gpg --armor --export-secret-keys 3AA5C34371567BD2 > chave-privada-BACKUP.asc
gpg --armor --export 3AA5C34371567BD2              > chave-publica.asc
```

Para restaurar em outra máquina:

```bash
gpg --import chave-privada-BACKUP.asc
```

### Certificado de revogação

O GnuPG 2.1 ou mais novo cria um automaticamente em
`~/.gnupg/openpgp-revocs.d/<fingerprint>.rev`. Guarde-o junto com o backup.

### Renovar antes de expirar

```bash
gpg --edit-key 3AA5C34371567BD2
gpg> expire          # muda a validade da chave principal
gpg> key 1           # seleciona a subchave
gpg> expire          # muda a validade da subchave
gpg> save
```

Depois de renovar, **exporte e recadastre** a chave pública no GitHub: apague
a antiga e adicione a nova. Commits assinados antes da expiração continuam
**Verified**.

---

Próximo: [3. Uso no dia a dia →](03-uso-diario.md)
