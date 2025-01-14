# Maratona Full Cycle - Codelivery - Part V - Driver - CI/CD

O projeto consiste em:

- Um sistema de monitoramento de veículos de entrega em tempo real.

Requisitos:

- Uma transportadora quer fazer o agendamento de suas entregas;
- Ela também quer ter o _feedback_ instantâneo de quando a entrega é realizada;
- Caso haja necessidade de acompanhar a entrega com mais detalhes, o sistema deverá informar, em tempo real, a localização do motorista no mapa.

#### Que problemas de negócio o projeto poderia resolver?

- O projeto pode ser adaptado para casos de uso onde é necessário rastrear e monitorar carros, caminhões, frotas e remessas em tempo real, como na logística e na indústria automotiva.

Dinâmica do sistema:

1. A aplicação _Order_ (_React_/_Nest.js_) é responsável pelas ordens de serviço (ou pedidos) e vai conter a tela de agendamento de pedidos de entrega. A criação de uma nova ordem de serviço começa o processo para que o motorista entregue a mercadoria;

2. A aplicação _Driver_ (_Go_) é responsável por gerenciar o contexto limitado de motoristas. Neste caso, sua responsabilidade consiste em disponibilizar os _endpoints_ de consulta;

3. Para a criação de uma nova ordem de serviço, a aplicação _Order_ obtém de _Driver_ os dados dos motoristas. Neste caso, _REST_ é uma opção pertinente, porque a comunicação deve ser a mais simples possível;

4. Após criar a nova ordem de serviço, _Order_ notifica a aplicação _Mapping_ (_Nest.js_/_React_) via _RabbitMQ_ de que o motorista deve iniciar a entrega. _Mapping_ é a aplicação que vai exibir no mapa a posição do motorista em tempo real. A aplicação _Simulator_ (_Go_) também é notificada sobre o início da entrega e começa a enviar para a aplicação _Mapping_ as posições do veículo;

5. Ao finalizar a entrega, a aplicação _Mapping_ notifica via _RabbitMQ_ a aplicação _Order_ de que o produto foi entregue e a aplicação altera o _status_ da entrega de Pendente para Entregue.

## Tecnologias

#### Operate What You Build

Nesta quinta versão, trabalhamos a parte de _Continuous Integration_ e _Continuous Deploy_. Iniciamos pela aplicação Driver.

- Backend

  - Golang

- CI (Continuous Integration)

  - GitHub Actions

- CD (Continuous Deploy)

  - Google Cloud Build

- Deploy
  - Kubernetes GKE

### O que faremos

O objetivo deste projeto é cobrir um processo de desenvolvimento do começo ao fim, desde:

- A utilização de uma metodologia para se trabalhar com o Git - o GitFlow;
- A adoção de Conventional Commits como especificação para as mensagens de commits;
- A adoção de Semantical Versioning (SemVer) como convenção para o versionamento de versões do software;

Até a definição de:

- Pipelines de Integração Contínua;
- Pipelines de Entrega Contínua;

Realizando, por fim:

- O deploy da aplicação em um cluster Kubernetes junto a um cloud provider.

Assim, seguimos uma seqüência de passos:

1. Definição da metodologia de Git Flow;

2. Definição de um nova Organização no GitHub;

3. Configuração de branches no GitHub;

- Filtros por branches;
- Proteção do branch master para evitar push direto;

4. Configuração de Pull Requests (PRs) no GitHub;

- Templates para PRs;

5. Configuração de Code Review no GitHub;

6. Configuração de CODEOWNERS no GitHub;

7. GitHub Actions;

8. Integração do SonarCloud Scan (Linter) ao GitHub Actions;

9. Definição de um cluster GKE utilizando Terraform;

10. Deploy da aplicação utilizando o Google Cloud Build integrado ao GitHub Actions;

### Definição da metodologia de Git Flow

- Qual é o problema que o Git Flow resolve?

Na verdade, o Git Flow é uma metodologia de trabalho que visa simplificar e organizar o processo que envolve o versionamento de código-fonte. Portanto, ele resolve alguns problemas, como, por exemplo:

- Como definir um branch para uma correção?
- Como definir um branch para uma funcionalidade nova?
- Como saber se uma funcionalidade nova ainda está em desenvolvimento ou foi concluída?
- Como saber se o que está no branch master é o mesmo que está em Produção?
- Como saber se existe um branch de desenvolvimento?

Vejamos, então, dois cenários simples do dia-a-dia aonde é empregado o Git Flow.

#### Cenário I

Neste cenário, há a definição de um branch master, também conhecido como o branch da verdade, porque, normalmente, o que está no master equivale ao que está em Produção.

O Git Flow recomenda que não se commit diretamente no branch master. O master não deve ser um local onde se consolidam todas as novas funcionalidades.

Então, o Git Flow define um branch auxiliar para que, quando todas as funcionalidades novas forem agregadas no branch auxiliar, aí sim elas são jogadas para o branch master. Esse branch auxiliar é chamado de _develop_.

O Git Flow define também um branch chamado de feature para desenvolver cada nova funcionalidade. Quando o desenvolvimento finaliza, é feito um merge no branch develop. Ou seja, ao finalizar, é tudo jogado para o branch develop. Frisando que nunca deve-se mergear diretamente uma funcionalidade nova com o branch master.

Sumarizando, o Git Flow define um branch para _feature_, _develop_ e _master_.

#### Cenário II

Neste cenário, há a definição do branch release. Ou seja, antes de colocar qualquer coisa no branch master, é necessário criar, antes, um branch de release.

E, a partir desse branch de release, é gerada uma tag e, depois, faz-se o merge no branch master.

Dessa forma, no dia-a-dia, tudo o que for sendo consolidado é jogado no branch develop. Porém, ao concluir uma _sprint_, por exemplo, pode-se separar as novas funcionalidades e colocá-las em um branch de release.

Uma vez que haja uma release, pode-se solicitar ao pessoal de QA, por exemplo, verificar se não há mais nada para corrigir.

Estando tudo correto, pode-se fazer o merge. E, a partir do merge, é gerada uma tag, que é enviada para o branch master.

### Definição de um nova Organização no GitHub

Antes de realizarmos o nosso primeiro _commit_ no _GitHub_, é necessário atentar que, para trabalhar no _GitHub_ de forma colaborativa, ou seja, aonde haja a participação de outros colaboradores nos repositórios, é necessário criar uma organização onde os colaboradores possam ser vinculados.

Nesse sentido, criamos uma organização fictícia, a _maratonafullcyclesidartaoss_ e vinculamos 3 usuários do GitHub a essa organização, conforme a definição abaixo:

![Nova organização e usuários do GitHub vinculados](./images/nova-organizacao-e-usuarios-vinculados.png)

Logo, para o commit da aplicação driver:

1. Criamos um novo repositório público na organização _maratonafullcyclesidartaoss_: o _fullcycle-maratona-1-codelivery-part-5-driver_.

![Criação do repositório Driver](./images/criacao-repositorio-driver.png)

2. E para dar o _push_ inicial da aplicação no _GitHub_:

```
git init

git add .

git commit -m "feat: add driver"

git remote add origin https://github.com/maratonafullcyclesidartaoss/fullcycle-maratona-1-codelivery-part-5-driver.git

git push -u origin master
```

### Continuous Integration

Esses são alguns dos principais subprocessos envolvidos na execução do processo de CI e que são cobertos neste projeto:

- Execução de testes;
- _Linter_;
- Verificação de qualidade de código;
- Verificação de segurança;
- Geração de artefatos prontos para o processo de _deploy_.

Algumas das ferramentas populares para a geração do processo de Integração Contínua:

- _Jenkins_;
- _GitHub_ _Actions_;
- _AWS CodeBuild_;
- _Azure DevOps_;
- _Google Cloud Build_;
- _GitLab CI/CD_.

#### GitHub Actions

A ferramenta escolhida para este projeto é o _GitHub_ _Actions_. Principalmente porque:

- É livre de cobrança para repositórios públicos;
- É totalmente integrada ao _GitHub_.

Estar integrado ao _GitHub_ pode ser considerado um diferencial, porque, baseado em eventos que acontecem no repositório, vários tipos de Ações (_Actions_), além das relacionadas ao processo de _CI_, podem ser executadas.

Sempre se inicia uma _GitHub Action_ a partir de um _workflow_.

#### Workflow

O _workflow_ consiste em um conjunto de processos definidos pelo desenvolvedor, sendo que é possível ter mais de um _workflow_ por repositório.

Um _workflow_:

- É definido em arquivos _.yaml_ no diretório _.github/workflows_;
- Possui um ou mais _jobs_ (que o _workflow_ roda);
- É iniciado a partir de eventos do _GitHub_ ou através de agendamento.

#### Eventos

Para cada evento, é possível definir filtros, ambiente e ações.

Exemplo:

- **Evento**:
  - _on: push_
- Filtros:
  - _branches_:
    _master_
- Ambiente:
  - _runs-on: ubuntu_
- Ações:
  - _steps_:
    - _uses: action/run-composer_
    - _run: npm run prod_

Nesse exemplo:

- É disparado um _evento_ de _on_ _push_, onde alguém executou um _push_ no repositório;
- O _ambiente_ define a máquina em que o processo de _CI_ deve rodar; nesse caso, em uma máquina _ubuntu_;
- O _filtro_ define que o evento deve acontecer somente quando for executado um _push_ para o _branch master_;
- As _ações_ definem passos (_steps_), subdivididos em duas opções:
  - _uses_ define uma _Action_ do _GitHub_, ou seja, um código desenvolvido por um desenvolvedor para ser executado no padrão do _GitHub Actions_. Inclusive, há um _marketplace_ do _GitHub Actions_ (`https://github.com/marketplace`);
  - _run_ permite executar uma _Action_ ou um comando; nesse caso, é executado um comando dentro da máquina _ubuntu_.

### Configuração de branches no GitHub

São consideradas boas práticas que dão mais segurança e tranqüilidade aos membros da equipe ao trabalhar com repositórios no GitHub:

- Nunca comitar diretamente no _branch master_;
- Nunca comitar diretamente no _branch develop_;
- Sempre trabalhar com _Pull Requests_.

Sendo assim, a seguir, são feitas algumas configurações básicas para a proteção dos _branches_.

### Protegendo branches

A princípio, o repositório é criado apenas com o _branch master_. Então, deve-se criar o _branch develop_ também:

```
git checkout -b develop

git push origin develop
```

![Criação branch develop](./images/criacao-branch-develop.png)

Dessa forma, o _branch master_ não será mais o _branch_ padrão. Ele será um _branch_ que utilizaremos como base para verificar se o que está nele equivale ao que está em Produção.

E o _branch_ padrão que utizaremos para comitar, conforme o processo de _Git Flow_, será o _develop_.

Neste momento, então, nós vamos nas configurações do _GitHub_, em _Settings_ / _Default branch_ e alteramos de _master_ para _develop_, para garantir que os _commits_ não serão feitos diretamente para o _branch master_.

Outra configuração importante refere-se à parte de proteção. Para isso, ao acessar _Settings / Branches / Branch protection rules_, é possível adicionar regras de proteção para os _branches_.

Primeiramente, faremos a proteção do _branch master_. Neste momento, nós iremos restringir o _push_ para alguns grupos ou pessoas, ao marcar a opção _Restrict who can push to matching branches_. Por ora, é isso. Clicamos em Create.

Em seguida, fazemos a proteção do _branch develop_, da mesma forma que fizemos com o _branch master_.

Agora que os branches estão minimamente protegidos, vamos iniciar com o processo de _Pull Requests_.

### Pull Requests

Primeiramente, nós criamos um novo branch para uma nova funcionalidade:

```
git checkout -b feature/update-readme
```

Vamos alterar o arquivo _README.md_ e adicionar um arquivo no diretório _images_. Então, damos um _commit_ e um _push_ para subir no _GitHub_:

```
git add .

git commit -m "docs: update readme.md"

git push origin feature/update-readme
```

O repositório no GitHub verifica que um novo branch chamado feature/update-readme subiu e já oferece a opção de comparar esse branch com os demais branches do repositório e já realizar uma _Pull Request_.

![Compare & pull request](./images/compare-and-pull-request.png)

Ao clicar em _Compare & pull request_, o GitHub mostra a opção de solicitar uma _Pull Request_ (_PR_) para o branch _develop_. Todas às vezes em que é criado um _PR_, é necessário detalhar sobre o que se trata o _PR_ na parte de comentários. Então, após deixar um comentário, clica-se em _Create pull request_.

A partir deste momento, caso não haja nenhum conflito impedindo, é possível realizar o _merge_ para _develop_. Então, é só clicar em _Merge pull request_ e _Confirm merge_.

O _GitHub_ aproveita para sugerir que deletemos o branch _feature/update-readme_, uma vez que já foi mergeado. Após deletar o _branch_ no _GitHub_, é necessário, também, deletar na máquina local:

```
git checkout develop

git pull origin develop

git branch

git branch -d feature/update-readme

git branch
```

### Criando Template para PRs

Toda vez em que é criado um _PR_, é possível adicionar um detalhamento na parte de comentários para esse _PR_.

No entanto, esse detalhamento pode deixar muito a desejar, porque pode ficar em um escopo muito aberto.

Por conta disso, é possível trabalhar com a utilização de _templates_ para _PRs_. Então, toda vez que for criado um novo _PR_, um _template_ pré-montado é apresentado à pessoa que esteja criando o _PR_ para que ela possa seguir algumas diretrizes.

Nesse sentido, vamos utilizar um _template_ baseado em um modelo disponível no _site_ _Embedded Artistry_: `https://embeddedartistry.com/blog/2017/08/04/a-github-pull-request-template-for-your-projects`.

A partir desse modelo, nós vamos criar um arquivo chamado _PULL_REQUEST_TEMPLATE.md_ dentro do diretório _.github_.

O _checklist_ de opções vai depender de cada projeto e das necessidades de cada equipe, mas, o mais importante é ter o modelo de _template_; a partir dele, é possível adaptar o _checklist_ às demandas de cada time.

Então, criamos o _template_ a partir de uma nova funcionalidade:

```
git checkout -b feature/pull-request-template

```

Criamos um diretório chamado _.github_ e, dentro desse diretório, o arquivo _PULL_REQUEST_TEMPLATE_.md\_.

```
mkdir .github

touch .github/PULL_REQUEST_TEMPLATE.md
```

Colamos nesse arquivo o conteúdo do _site_ _Embedded Artistry_. Em seguida, comitamos e subimos para o _GitHub_.

```
git add .

git commit -m "chore: add pull request template"

git push origin feature/pull-request-template
```

#### Referência

FULL CYCLE 3.0. Integração contínua. 2023. Disponível em: <https://plataforma.fullcycle.com.br>. Acesso em: 26 mai. 2023.

FULL CYCLE 3.0. Padrões e técnicas avançadas com Git e Github. 2023. Disponível em: <https://plataforma.fullcycle.com.br>. Acesso em: 26 mai. 2023.
