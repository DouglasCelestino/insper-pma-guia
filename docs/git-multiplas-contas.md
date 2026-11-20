# Várias contas Git na mesma máquina

São dois problemas separados:

- **Identidade:** nome e e-mail gravados nos commits. Resolvida com `includeIf`, por pasta.
- **Login:** conta usada em `clone` e `push`. Resolvido com o usuário na URL, via `url.<base>.insteadOf`, e o Git Credential Manager.

## Modelo para uma máquina nova

`~/.gitconfig`:

```ini
[user]
    useConfigOnly = true

[init]
    defaultBranch = main

[includeIf "gitdir/i:~/Documents/Insper/"]
    path = ~/.gitconfig-pessoal

[includeIf "gitdir/i:~/Documents/Empresa/"]
    path = ~/.gitconfig-empresa

[url "https://SEU_USUARIO@github.com/insper-plat-projeto/"]
    insteadOf = https://github.com/insper-plat-projeto/
```

`~/.gitconfig-pessoal` (e o equivalente para a empresa):

```ini
[user]
    name = SEU_USUARIO
    email = seu@email.com
```

- A barra final em `gitdir/i:.../` inclui todas as subpastas. O `/i` ignora maiúsculas e minúsculas.
- `useConfigOnly` faz o Git recusar o commit quando não há identidade definida. Só funciona sem `name` e `email` globais.
- O bloco `[url]` fica no arquivo global para valer durante o `clone`. Repita-o para cada organização ou conta.
- Tokens e senhas não entram nesses arquivos.

## Como testar

```
git config --show-origin --get user.email
git remote get-url origin
```

O primeiro mostra o e-mail e o arquivo de onde ele veio. O segundo deve mostrar `SEU_USUARIO@` na URL.

## Problema conhecido: `gh auth setup-git`

O GitHub CLI pode registrar `gh auth git-credential` como helper de `https://github.com` no `~/.gitconfig`. Ele só entrega o token da conta **ativa** no `gh`. Com outra conta ativa, o Git falha assim:

```
fatal: could not read Password for 'https://SEU_USUARIO@github.com': No such file or directory
```

Diagnóstico:

```
gh auth status
git config --show-origin --get-all credential.https://github.com.helper
```

Soluções:
- Rápida, a cada troca: `gh auth switch -u SEU_USUARIO`.
- Permanente: mandar só a conta pessoal para o Git Credential Manager. A regra com usuário na URL é mais específica e vence a do `gh`:
  ```
  git config --global --add "credential.https://SEU_USUARIO@github.com.helper" ""
  git config --global --add "credential.https://SEU_USUARIO@github.com.helper" manager
  ```
  O valor vazio limpa os helpers herdados antes de definir o `manager`.

## Exceções

- Repos pessoais fora da organização: clone com `https://SEU_USUARIO@github.com/SEU_USUARIO/repo.git`.
- URLs `git@github.com:...` usam SSH, e essa regra não se aplica a elas.

## Onde pesquisar

- `git help config`, seções `includeIf` e `url.<base>.insteadOf`
- `git help gitcredentials`
- Documentação do Git Credential Manager, seção "multiple users"
- Buscas: `git different email per directory`, `git credential manager multiple github accounts`
