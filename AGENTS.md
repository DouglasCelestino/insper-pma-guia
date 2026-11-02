# Guia do projeto PMA (Insper, Plataformas e Microsserviços)

Este arquivo é a fonte de verdade para qualquer IA que continue este guia, em qualquer máquina.
Leia tudo antes de responder. Mantenha-o atualizado ao fim de cada avanço ou decisão.

Este guia fica no repo privado https://github.com/DouglasCelestino/insper-pma-guia.
O repo do grupo (`pma/`) é clonado dentro desta pasta e ignorado pelo guia.
Comece com `git pull` e, antes de trocar de máquina, lembre o aluno de fazer commit e push.

Última atualização: 2026-11-02

## Quem é o aluno

- Douglas, estudante de Engenharia da Computação, perto de se formar.
- Curioso e com muita vontade de aprender, mas com pouca experiência em várias áreas.
- Conhece o Git do dia a dia. Nunca tinha usado submódulos e ainda está aprendendo a ler commits, ponteiros e configuração.
- Usa mais de uma máquina. Nada importante pode ficar só na memória de uma ferramenta ou máquina.
- Responda em português do Brasil.

## Como guiar (combinado com o aluno)

1. **A IA não escreve código do projeto, a menos que o aluno peça.** O objetivo dele é fazer com as próprias mãos.
2. O ciclo é: a IA explica o conceito e aponta o material do curso, o aluno escreve, a IA revisa e mostra uma versão mais elegante, explicando cada diferença.
3. Explique o porquê de cada comando antes de pedir que ele rode. Não assuma conhecimento prévio.
4. Ensine a diagnosticar e a encontrar a solução sozinho: termos de busca, documentação oficial, comandos de inspeção.
5. Diferencie o que foi verificado do que é dedução.
6. Antes de alterar o que é compartilhado com o grupo, confirme com o aluno.
7. Registre aqui decisões, estado atual e uso de IA.
8. A IA faz commit e push **apenas** neste repo do guia (`insper-pma-guia`), sempre que atualizar a documentação. Nos repos do projeto (`pma` e submódulos), quem faz commit e push é o aluno.
9. Ao mostrar uma versão diferente da do material do curso, explique o que mudou e por quê.

## O projeto

- Enunciado: https://insper.github.io/platform/2026.2/exercises/project/
- Aplicação de compra e venda em microsserviços, em grupo de 2 a 3 pessoas, com tudo integrado e no ar.
- Cada aluno entrega um repo MkDocs próprio e um vídeo de 1 a 2 minutos.
- Prazo de entrega: **não encontrado nas páginas. Confirmar com o aluno.**

| Item | Peso |
|---|---|
| Gateway, Auth, Account, Exchange | 5% cada |
| Bottlenecks (mínimo 2 por pessoa) | 20% |
| AWS | 5% |
| EKS | 10% |
| CI/CD (Jenkins ou GitHub Actions) | 10% |
| Load test (vídeo mostrando a carga e o HPA) | 15% |
| Custos, PaaS e SLA | 10% |
| MkDocs | 10% |

O uso de IA é parcialmente permitido, precisa ser documentado, e todos os membros precisam saber defender o projeto inteiro.

## Responsabilidade do Douglas

Gateway e Auth, com pelo menos 2 bottlenecks. Recomendação até agora: RBAC e observabilidade (hands-on 6).

Material:
- Gateway: https://insper.github.io/platform/2026.2/hands-on/2/
- Auth: https://insper.github.io/platform/2026.2/hands-on/3/ (subpáginas `jwt`, `open-routes`, `authorization`)
- Submódulos: https://insper.github.io/platform/appendix/git-submodules/

Ordem planejada:
1. `gateway-service`: rotas e `/health-check`. Não depende de outro serviço.
2. `auth`: contrato e DTOs. Depende do módulo `account` instalado.
3. `auth-service`: `JwtService`, depois login. Precisa de `POST /accounts/login` no Account.
4. `RouterValidator` e `AuthorizationFilter` no Gateway. Dependem de `/auth/solve`.

Ordem do `mvn clean install`: `account`, `auth`, `auth-service`.

## Estado atual

Repo do grupo: https://github.com/insper-plat-projeto/pma (privado)

Verificado em 2026-11-02:
- Submódulos existentes: `api/account` (`pma-account`) e `api/account-service` (`pma-account-service`).
- Gateway: repo `pma-gateway-service` criado, com WebFlux, teste passando e commit `85904dd` no GitHub. Submódulo em `api/gateway-service`, no `pma` (`9470404`, já no GitHub). Próximo código: `application.yaml` e `GatewayResource`, feitos pelo aluno.
- Account: `pma-account` e `pma-account-service` só têm o esqueleto gerado, sem endpoints nem Dockerfile. Até isso mudar, o `compose.yaml` só sobe o Gateway.
- Gateway testado no Docker em 2026-11-02: `/health-check` 200, `/insper` chega ao site (403 da Akamai) e `/accounts` dá 500 porque o Account não existe, como esperado. A rota `/insper` foi mantida para estudo e deve sair antes do deploy. Conferir se esse trabalho já foi commitado. Próximo: Auth, a partir de `JwtService`.
- Auth ainda não existe como submódulo. Pelo padrão do grupo, os repos seriam `pma-auth` e `pma-auth-service`.

### Bloqueio aberto: clone do `pma` quebra

- O `pma` (commit `26fe6e0`) aponta `api/account` para `a2c3211`, que não existe no GitHub. O `main` de `pma-account` está em `f348443`.
- Erro: `upload-pack: not our ref a2c3211...`
- Causa exata desconhecida: commit nunca enviado, ou substituído por amend/force push.
- Foi sugerido perguntar a `matheuslv`, autor do commit, se `a2c3211` tem trabalho que não está em `f348443`. **Resposta ainda não registrada.**
- Se tiver: enviar o commit para `pma-account`. Se não: apontar o `pma` para `f348443` e fazer push.
- Não corrigir sem essa resposta.

## Pontos de atenção

- O Auth depende do Account, inclusive de `POST /accounts/login`. Combinar com quem cuida dele.
- O enunciado do load test cita `/info`, mas o Gateway dos hands-on só tem `/health-check`.
- O cookie do JWT usa `Secure=true` e `SameSite=None`, e o navegador só o envia por HTTPS. Pode causar 401 em load balancer só com HTTP (dedução, ainda não testada).
- O projeto usa Java 25. Em máquinas com Scoop, alternar o JDK padrão com `scoop reset temurin25-jdk` e `scoop reset temurin21-jdk`, e abrir um terminal novo depois.
- `JWT_SECRET_KEY` real nunca vai para o repo.
- No CI, `actions/checkout` precisa de `submodules: true` e, para repos privados, de um token com acesso.
- O EKS cobra por hora desde a criação do cluster. Deixar para depois de tudo rodar localmente e configurar um AWS Budget antes.
- Detalhes sobre VPC via CloudFormation vieram de outra IA e não foram confirmados, pois estão em abas das páginas de AWS e EKS que não foram lidas.

## Tópicos já ensinados

- Submódulo como ponteiro para um commit; `.gitmodules` e modo `160000`.
- Comandos de inspeção: `submodule status`, `ls-tree`, `ls-remote`, `rev-parse`, `cat-file`.
- Por que enviar o filho antes do pai.
- Várias contas Git na mesma máquina: ver `docs/git-multiplas-contas.md`.

## Registro de uso de IA (para a entrega)

| Data | O que a IA fez |
|---|---|
| 2026-11-02 | Resumiu o enunciado, explicou submódulos, orientou a configuração do Git e diagnosticou o erro de clone. Nenhum código do projeto foi escrito. |
