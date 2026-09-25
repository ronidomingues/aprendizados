# 6. Reutilizando a chave GPG em outra máquina

[← Voltar ao índice](README.md)

Uma chave GPG não pertence a uma máquina: ela pertence a **você**. Levar a
mesma chave para outro computador é normal, e o GitHub não precisa de nenhuma
mudança, porque ele reconhece a chave e não o computador.

> 💡 **Alternativa avançada:** em vez de copiar a chave principal, dá para
> criar uma **subchave de assinatura por máquina** e levar só ela. Assim, se um
> computador for comprometido, você revoga só a subchave dele. Este capítulo
> cobre o caso simples, que é a mesma chave em todas as máquinas.

## 6.1 Na máquina de origem: exportar

```bash
ID=3AA5C34371567BD2                            # o ID da sua chave (veja 2.4)
FPR=$(gpg --with-colons --fingerprint $ID | awk -F: '/^fpr/ {print $10; exit}')

gpg --armor --export-secret-keys $ID > chave-privada.asc
gpg --armor --export             $ID > chave-publica.asc
gpg --export-ownertrust | grep $FPR   > confianca.txt
cp ~/.gnupg/openpgp-revocs.d/$FPR.rev  revogacao.rev
chmod 600 chave-privada.asc revogacao.rev
```

| Arquivo | Para que serve | Obrigatório? |
| :-- | :-- | :-: |
| `chave-privada.asc` | Contém a chave privada e as subchaves. É o que permite assinar. | ✅ |
| `chave-publica.asc` | A parte pública. Já vem dentro da privada, mas é útil ter separada. | — |
| `confianca.txt` | O nível de confiança (`[ultimate]`). Sem ele, a chave aparece como `[unknown]`. | Recomendado |
| `revogacao.rev` | Permite revogar a chave se ela for comprometida. | Guarde no backup |

A chave privada exportada **continua protegida pela senha da chave**. Para
confirmar:

```bash
gpg --list-packets chave-privada.asc | grep -m1 protected
```

Se aparecer `protected`, sem a senha ninguém consegue usá-la.

## 6.2 Transportar os arquivos

| Meio | Situação |
| :-- | :-- |
| `scp` ou `rsync` por SSH entre as duas máquinas | ✅ Bom |
| Pendrive seu, apagando o arquivo depois | ✅ Bom |
| Repositório **privado** de backup, só seu | ✅ Aceitável, porque o arquivo está protegido pela senha |
| E-mail, chat, nuvem compartilhada | ❌ Evite |

Exemplo com `scp`:

```bash
scp chave-privada.asc confianca.txt usuario@outra-maquina:~/
```

## 6.3 Na máquina de destino: importar

```bash
gpg --import chave-privada.asc             # pede a senha da chave
gpg --import-ownertrust confianca.txt
gpg --list-secret-keys --keyid-format=long
```

O resultado deve mostrar:

```
sec   rsa4096/3AA5C34371567BD2 ... [SC] [expires: ...]
uid                 [ultimate] Seu Nome <seu-email@exemplo.com>
ssb   rsa4096/... [E]
```

> ⚠️ Se aparecer **`sec#`** (com `#`), só a parte pública foi importada, sem a
> chave privada. Refaça a exportação com `--export-secret-keys`, e não com
> `--export`.

Depois de importar, apague os arquivos transportados. Eles não são mais
necessários, e o original continua no backup:

```bash
shred -u chave-privada.asc confianca.txt
```

## 6.4 Configurar o Git na máquina de destino

```bash
git config --global user.name  "Seu Nome"
git config --global user.email seu-email@exemplo.com   # o mesmo da chave
git config --global gpg.format openpgp
git config --global user.signingkey 3AA5C34371567BD2
git config --global commit.gpgsign true
git config --global tag.gpgsign true
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc && source ~/.bashrc
```

## 6.5 Particularidades de cada sistema

### Debian, Ubuntu e Kali

```bash
sudo apt install gnupg git
```

Depois disso, os passos acima funcionam sem mudança.

### Windows

1. Instale o **[Gpg4win](https://www.gpg4win.org/)**.
2. Importe a chave pelo **Kleopatra** (*Importar…*) ou pelo terminal, com os
   mesmos `gpg --import` do passo 6.3.
3. O Git for Windows traz o próprio `gpg`, que **não enxerga** o chaveiro do
   Gpg4win. Aponte o Git para o GPG certo:
   ```powershell
   where.exe gpg          # mostra onde o Gpg4win instalou o gpg.exe
   git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
   ```
   Use o caminho que o `where.exe` mostrar.
4. Não precisa do `GPG_TTY`: a senha é pedida numa janela.

### macOS

```bash
brew install gnupg pinentry-mac
echo "pinentry-program $(brew --prefix)/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

O `pinentry-mac` pede a senha numa janela e pode guardá-la no Keychain.

### Android (Termux)

```bash
pkg install gnupg git
```

Se a senha não for pedida, ou se o commit falhar com `gpg failed to sign the
data`, ative a entrada de senha pelo próprio terminal:

```bash
echo "allow-loopback-pinentry" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
git config --global gpg.program "gpg --pinentry-mode loopback"
```

## 6.6 Testar

```bash
echo teste | gpg --clearsign > /dev/null && echo OK

cd /tmp && git init teste-gpg && cd teste-gpg
git commit --allow-empty -m "Testando a chave nesta máquina"
git log --format='%G? %GK' -1        # esperado: G 3AA5C34371567BD2
cd .. && rm -rf teste-gpg
```

## 6.7 Mantendo as máquinas em sincronia

### Depois de renovar a chave

A validade da chave fica registrada na **parte pública**. Depois de renovar
numa máquina (veja [2.8](02-gpg.md#renovar-antes-de-expirar)), leve a chave
pública atualizada para as outras:

```bash
# Na máquina onde renovou
gpg --armor --export 3AA5C34371567BD2 > chave-publica-renovada.asc

# Em cada uma das outras máquinas
gpg --import chave-publica-renovada.asc
gpg --list-keys 3AA5C34371567BD2        # confira a data nova em [expires: ...]
```

Não é preciso reimportar a chave privada, porque ela não muda. Se uma máquina
ficar com a data antiga, ela **para de assinar** quando essa data chegar.

Atualize também o backup e o cadastro no GitHub (veja
[2.8](02-gpg.md#renovar-antes-de-expirar)).

### Tirar a chave de uma máquina

Ao vender, formatar ou devolver um computador:

```bash
gpg --delete-secret-and-public-keys <FINGERPRINT>
git config --global --unset user.signingkey
```

Isso só apaga a cópia daquela máquina. A chave continua valendo nas outras e
no GitHub.

### Se uma máquina for comprometida

Como a chave é a mesma em todas, ela ficou comprometida em **todas**:

1. Revogue a chave com o certificado de revogação (veja
   [2.8](02-gpg.md#certificado-de-revogação)).
2. Apague a chave em **GitHub → Settings → SSH and GPG keys**, para ninguém
   conseguir mais commits **Verified** com ela.
3. Crie uma chave nova, cadastre no GitHub e repita este capítulo em cada
   máquina.

---

Próximo: [7. Reutilizando a chave SSH →](07-reutilizando-ssh.md)
