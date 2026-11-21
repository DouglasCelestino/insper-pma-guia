# Guia do projeto PMA (Insper, Plataformas e Microsserviços)

Este arquivo é a fonte de verdade para qualquer IA que continue este guia, em qualquer máquina.
Leia tudo antes de responder. Mantenha-o atualizado ao fim de cada avanço ou decisão.

Este guia fica no repo privado https://github.com/DouglasCelestino/insper-pma-guia.
O repo do grupo (`pma/`) é clonado dentro desta pasta e ignorado pelo guia.
Comece com `git pull` e, antes de trocar de máquina, lembre o aluno de fazer commit e push.

Última atualização: 2026-11-20

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
10. Método de estudo recomendado para código pronto do curso: ler e prever o que faz, escrever de memória, comparar com o original e explicar sem consultar. Código repetitivo (imports, `pom.xml`, DTOs simples) pode ser copiado. Partes centrais (`JwtService`, `AuthService`, `RouterValidator`, `AuthorizationFilter`) devem ser escritas assim.

## O projeto

- Enunciado: https://insper.github.io/platform/2026.2/exercises/project/
- Aplicação de compra e venda em microsserviços, em grupo de 2 a 3 pessoas, com tudo integrado e no ar.
- Cada aluno entrega um repo MkDocs próprio e um vídeo de 1 a 2 minutos.
- Prazo de entrega: **5 de dezembro de 2026**, informado pelo aluno em 2026-11-02.

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

Decisão para o Auth: começar por `auth`, sem a dependência de `account` e sem `GET /auth/me`, enquanto o Account não existir. O `pma-account` (`f348443`) está em `0.0.1-SNAPSHOT`, ainda tem classe principal e não tem `AccountOut`; o curso espera `1.0.0`, sem classe principal. Quando o Account estiver pronto, adicionar a dependência e o `me`. O `AuthController` do hands-on confirma que `me` retorna `AccountOut`, de `store.account`.

Plano proposto até a entrega (metas de início de semana, ajustar com o grupo):
- 2/11: `auth` e `JwtService`
- 9/11: Account pronto, login e registro, filtro do Gateway, Compose completo
- 16/11: bottlenecks e CI/CD
- 23/11: EKS, deploy e load test
- 30/11: custos, MkDocs e vídeo, com folga para a entrega

## Onde paramos (atualizado em 2026-11-20)

O aluno ficou de 2/11 a 20/11 sem avançar, e o projeto está atrasado em relação ao plano acima. Enquanto isso, `matheuslv` integrou ao `pma` (`7c4bff3`): Account com login e PostgreSQL, Exchange (Python), Redis como cache do Account, front-end web com nginx, Prometheus e Grafana.

Verificado no GitHub em 2026-11-20:
- `pma-account` `main` em `99f4457`: biblioteca `store:account:1.0.0`, igual ao contrato do curso, com `POST /accounts/login`. **O Auth está destravado** e não precisa mais de stub.
- `pma-auth` em `5e20e15`, sem o `me`. `pma-auth-service`: `main` em `ddcc2ec` (esqueleto) e trabalho em `feat/auth-service` (`1dd9210`). `pma-gateway-service` em `52bdb46`.
- O `compose.yaml` do grupo não tem `gateway` nem `auth`, e o `account` publica `8080:8080` direto. Isso passa por fora do Gateway (o `id-account` pode ser forjado) e conflita com a porta do Gateway. Combinar com o grupo antes de mudar.
- O Gateway não tem rota para o Exchange. O Prometheus tem jobs comentados para Gateway e Auth.
- Observabilidade e cache já foram feitos pelo colega. Os 2 bottlenecks do Douglas precisam ser outros, a combinar com o grupo.

Credenciais nesta máquina: o `gh` passou a ser o helper do `github.com` e só entrega o token da conta ativa (`DouglasPixcards`). Resolvido com `credential.https://DouglasCelestino@github.com.helper = manager` (ver `docs/git-multiplas-contas.md`).

Feito em 2026-11-20 pelo aluno: `pma` atualizado; `account` instalado com o wrapper do `auth` (o repo do Account não tem `mvnw`; avisar o colega); `me` reativado no `auth` (`e94c212`); `auth-service` compilou no repo real e foi para a `main` (`1dd9210`); `pma` em `18bf0ea`, com o submódulo `api/auth-service` e o ponteiro do `auth`. Erro repetido: `submodule add` rodado de dentro de `api/` criou `api/api/auth-service`; foi desfeito antes do push.

## Reta final (2026-11-20)

O aluno informou às 11h42 que a entrega é **hoje, 2026-11-20, às 23h59** (antes registrado como 5/12) e pediu que a IA passe a escrever o código para acelerar. A regra 1 fica suspensa até a entrega: a IA escreve, o aluno roda, revisa, faz commit e push, e precisa saber explicar cada parte. Todo código desta fase deve constar no registro de uso de IA.

Decisões: ninguém do grupo fez AWS, EKS, CI/CD nem load test. Bottlenecks do Douglas: load balancing (nginx na frente de réplicas do Gateway, padrão do hands-on 7) e vulnerability scanning (Trivy).

Plano: 1) integração no Compose e login pelo site; 2) bottlenecks; 3) CI/CD com GitHub Actions; 4) EKS, HPA e load test, se der tempo; 5) MkDocs individual, uso de IA e vídeo.

Bloco 1 e LB, escritos pela IA e sem commit: rota `/exchanges/**` e log `store` em `info` no Gateway; no Compose, `auth`, `gateway` com 2 réplicas sem porta, `lb` (nginx) em `8080:80` e `account` sem porta; `setup/nginx/nginx.conf`; `api/.env` local com `VOLUME_DB=./postgres-data`.

Próximos passos (comandos no `pma` são do aluno):
1. No `pma`: `gateway` e `auth` no Compose, rota do Exchange no Gateway e jobs do Prometheus. Discutir com o grupo a porta do `account`.
2. Pendências de estudo: explicar `generate` e `getId` do `JwtService` e o fluxo do login até `id-account`.

## Estado atual

Repo do grupo: https://github.com/insper-plat-projeto/pma (privado)

Verificado em 2026-11-02:
- Submódulos existentes: `api/account` (`pma-account`) e `api/account-service` (`pma-account-service`).
- Gateway: repo `pma-gateway-service` criado, com WebFlux, teste passando e commit `85904dd` no GitHub. Submódulo em `api/gateway-service`, no `pma` (`9470404`, já no GitHub). Próximo código: `application.yaml` e `GatewayResource`, feitos pelo aluno.
- Account: `pma-account` e `pma-account-service` só têm o esqueleto gerado, sem endpoints nem Dockerfile. Até isso mudar, o `compose.yaml` só sobe o Gateway.
- Gateway testado no Docker em 2026-11-02: `/health-check` 200, `/insper` chega ao site (403 da Akamai) e `/accounts` dá 500 porque o Account não existe, como esperado. A rota `/insper` foi mantida para estudo e deve sair antes do deploy. Enviado ao GitHub: gateway em `cfc3c06`, `pma` em `7f3b00b`. Fase 1 do Gateway (rotas e `/health-check`) concluída. Pendências do Gateway: `RouterValidator` e `AuthorizationFilter` (dependem de `/auth/solve`), rota `/auth/**`, retirada de `/insper` e endpoint do load test. Próximo: Auth, a partir de `JwtService`.
- Auth: biblioteca `pma-auth` concluída em `1.0.0`, com DTOs e `AuthController`, sem o `me`. No GitHub em `5e20e15`, com o `pma` apontando para ela em `e9faa80`. `pma-auth-service` criado (`ddcc2ec`) e em `api/auth-service`, com o código do hands-on digitado localmente, sem commit e sem compilar. Bloqueios: falta `store:account:1.0.0` (stub), os arquivos estão em `store.auth_service` em vez de `store.auth`, há duas classes principais e o `me` está comentado no `auth`. O aluno digitou do handout, então ainda precisa estudar o `JwtService` conforme a regra 10.
- Avaliação das 5 perguntas sobre o `AuthController` (2026-11-02): revisar com ele o papel da interface em Java (ele achou que servia para qualquer linguagem), quem coloca o `id-account` (ele respondeu o navegador; é o Gateway) e por que `login` retorna `Void` (ele esperava `TokenOut`; o token vai em cookie `HttpOnly`). Ele deve explicar esse fluxo de memória ao implementar o login e o filtro.
- Substituto local do Account (decisão de 2026-11-02, enquanto o colega não entrega): pasta `account-stub-temp/`, fora do `pma`, com `account` (cópia exata do contrato do curso, em `store:account:1.0.0`, incluindo `POST /accounts/login` da página `hands-on/3/open-routes`) e `account-service` (dados em memória, com Dockerfile). Sobe por `pma/api/compose.override.yaml`, listado em `pma/.git/info/exclude`. Nada disso vai para o grupo nem para o deploy, e não prova a integração com o Account real.
- Atenção no `pma`: `api/account` aparece modificado localmente (`f348443`, enquanto o `pma` aponta para `a2c3211`) e há alterações locais em `account-service` e `gateway-service`. Usar `git add api/auth`, nunca `git add .` nem `commit -a`, para não decidir o bloqueio do Account por acidente.
- Lembrete de erro recorrente: usar `git submodule add <url> api/<nome>` e `git mv`, nunca `mv`, para mover submódulos.

### Bloqueio do clone: resolvido

O `pma` apontava `api/account` para `a2c3211`, que não existia no GitHub. Resolvido pelo grupo até 2026-11-20: o `main` aponta para `99f4457`, que existe.

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

## Entrega (2026-11-20, 23h26)

- Compose: gateway (2 réplicas) atrás do nginx `lb` em 8080, auth, healthcheck do Postgres. Testado de ponta a ponta.
- Jenkins em `pma/jenkins` (localhost:9080), 4 jobs criados pela IA em disco, credenciais `dockerhub-credential` (`dody16`) e `github-credential` (token classic `repo`). Builds verdes: account #1, auth #1, auth-service #1, gateway-service #4. Trivy com cache em volume `trivy-cache` e `--timeout 15m`.
- EKS `store` em us-east-1 na conta do grupo `992382635549` (perfil `pma`, SSO `d-9067fdf7f1`). Deploy com `pma/k8s/deploy.sh`; HPA testado (1 → 8 pods, CPU 391%/50%). Exchange dá 502 na AWS (API externa). **Cluster precisa ser apagado** se ainda existir.
- Docs: `pma/docs` e site público https://douglascelestino.github.io/pma-docs/ (repo `DouglasCelestino/pma-docs`, publicado com `mkdocs gh-deploy`).
- Sem vídeo do load test nem vídeo individual (decisão do aluno).
