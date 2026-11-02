# Várias contas Git na mesma máquina

São dois problemas separados:

- **Identidade:** nome e e-mail gravados nos commits. Resolvida com `includeIf`, por pasta.
- **Login:** conta usada em `clone` e `push`. Resolvido com o usuário na URL, via `url.<base>.insteadOf`, e o Git Credential Manager.

## Modelo para uma máquina nova

`~/.gitconfig`:

```ini
[user]
    useConfigOnly = true

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

## Exceções

- Repos pessoais fora da organização: clone com `https://SEU_USUARIO@github.com/SEU_USUARIO/repo.git`.
- URLs `git@github.com:...` usam SSH, e essa regra não se aplica a elas.

## Onde pesquisar

- `git help config`, seções `includeIf` e `url.<base>.insteadOf`
- `git help gitcredentials`
- Documentação do Git Credential Manager, seção "multiple users"
- Buscas: `git different email per directory`, `git credential manager multiple github accounts`
