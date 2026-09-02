# Demo: escopo correto, contexto menor

Este repositório prepara a demonstração de **custom instructions** do Módulo 2.
Ele usa a aplicação Training Catalog do
[`01-lab`](https://github.com/impacta-ghcp-eng-moderna/01-lab/tree/main/src)
para mostrar que regras gerais e regras específicas podem ser combinadas sem
repetir todo o contexto em cada prompt.

## O que existe no repositório

O código fica em `src` e contém uma solução .NET 10 com:

| Projeto | Responsabilidade |
| --- | --- |
| `Api` | ASP.NET Core Minimal API |
| `Application` | Contratos do domínio |
| `Infrastructure` | Entity Framework Core e SQLite |
| `Client` | Blazor WebAssembly |
| `Tests/Api.Tests` | Testes funcionais xUnit |

Além da aplicação, o repositório contém um Codespace mínimo e quatro arquivos
que orientam agentes de IA em escopos diferentes:

```text
.
|-- .devcontainer/devcontainer.json
|-- .github/copilot-instructions.md
|-- .github/instructions/
|   |-- api.instructions.md
|   `-- tests.instructions.md
|-- AGENTS.md
`-- src/
```

## Codespace mínimo

`.devcontainer/devcontainer.json` usa a imagem oficial
`mcr.microsoft.com/devcontainers/dotnet:1-10.0-noble`. Somente as extensões
GitHub Copilot e GitHub Copilot Chat, necessárias para a demonstração, são
instaladas. Nenhuma Feature, porta ou instalação de `sqlite3` foi configurada.
O runtime nativo usado pelo pacote do Entity Framework Core já é suficiente
para executar a aplicação e os testes; o programa de linha de comando `sqlite3`
não é necessário para esta demonstração.

Depois de criar o Codespace, confirme o ambiente e a solução:

```bash
dotnet --version
dotnet restore src/TrainingCatalog.slnx
dotnet test src/TrainingCatalog.slnx --no-restore
```

A primeira saída deve começar com `10.`.

## Como os arquivos de instructions funcionam

### `.github/copilot-instructions.md`

É a instruction de todo o repositório. O VS Code a inclui automaticamente em
todas as solicitações de chat neste workspace. Ela registra apenas decisões
abrangentes: estrutura da solução, versão do .NET, idioma dos termos e
mensagens, tamanho das mudanças e validação.

Ela também pede que resumos de alterações comecem com `GERAL:`. Esse marcador
é propositalmente visível para a aula, mas a evidência mais confiável continua
sendo a lista de referências ou customizações carregadas exibida pelo chat.

Esse nome de arquivo tem significado especial, portanto ele não depende de
front matter nem de `applyTo`.

### `.github/instructions/api.instructions.md`

Contém somente regras da ASP.NET Core Minimal API: rotas, contratos de erro,
status documentados e acesso assíncrono ao Entity Framework Core. Seu front
matter possui os três campos apresentados na demonstração:

| Campo | Valor nesta demo | Efeito |
| --- | --- | --- |
| `name` | `API do Training Catalog` | Nome mostrado na interface |
| `description` | Resumo das convenções da API | Ajuda o agente a identificar a finalidade |
| `applyTo` | `src/Api/**/*.cs` | Aplica automaticamente as regras ao trabalhar em C# dentro de `src/Api` |

Quando aplicado, esse arquivo também pede uma linha iniciada por `API:`.

### `.github/instructions/tests.instructions.md`

Contém somente regras dos testes funcionais: xUnit,
`TrainingCatalogApiFactory`, chamadas HTTP, isolamento do SQLite, nomenclatura
e ordem das asserções. Seu `applyTo` é `src/Tests/**/*.cs`, portanto essas
regras não ocupam o contexto de uma alteração restrita à API.

Quando aplicado, esse arquivo pede uma linha iniciada por `TESTES:`.

### `AGENTS.md`

É um formato interoperável reconhecido por diferentes agentes. Neste
repositório, ele fornece o mapa curto dos projetos, os comandos de validação e
protege migrations e o banco de alterações acidentais. O arquivo na raiz é
considerado em todo o workspace e não usa front matter.

O suporte a `AGENTS.md` pode ser controlado pela configuração
`chat.useAgentsMdFile`. Arquivos `AGENTS.md` aninhados também são possíveis,
mas esse comportamento é experimental e não é necessário nesta demo.

## Composição e seleção de contexto

O VS Code pode combinar mais de uma instruction na mesma solicitação, sem
garantir uma ordem entre elas. Por isso, os arquivos deste repositório têm
responsabilidades complementares e não contêm regras contraditórias.

- `.github/copilot-instructions.md` e `AGENTS.md` são gerais e automáticos.
- `api.instructions.md` entra quando a tarefa trabalha com arquivos que
  correspondem a `src/Api/**/*.cs`.
- `tests.instructions.md` entra quando a tarefa trabalha com arquivos que
  correspondem a `src/Tests/**/*.cs`.
- Um arquivo `*.instructions.md` também pode ser anexado manualmente, mesmo
  quando o arquivo atual não corresponde ao seu `applyTo`.

As custom instructions afetam o chat e os agentes, mas não as sugestões
inline mostradas enquanto se digita no editor.

## Preparação da demonstração

1. No VS Code, execute **Chat: Open Customizations** pela Command Palette.
2. Abra a aba **Instructions** e localize os arquivos por origem e escopo.
3. Abra `api.instructions.md` e `tests.instructions.md` e destaque `name`,
   `description` e `applyTo`.
4. Abra o chat no modo Agent.
5. Em cada resposta, expanda as referências usadas pelo chat. Os marcadores
   textuais facilitam a visualização, mas as referências comprovam qual
   instruction foi carregada.

Se a interface não atualizar uma instruction recém-editada, inicie uma nova
conversa antes de repetir o prompt.

## Prompts para testar cada comportamento

### 1. Instructions gerais

Com `src/Application/Training.cs` aberto, envie:

```text
Sem alterar arquivos, explique como você implementaria uma nova propriedade
opcional de treinamento e quais projetos da solução seriam afetados.
```

Observe `GERAL:` e as referências a `.github/copilot-instructions.md` e
`AGENTS.md`. As instructions de API e testes não devem ser necessárias.

### 2. Escopo da API

Com `src/Api/Program.cs` aberto, envie:

```text
Adicione um endpoint GET /api/trainings/count que retorne a quantidade de
treinamentos cadastrados. Mantenha os contratos e padrões existentes.
```

Observe `GERAL:`, `API:` e a referência a `api.instructions.md`. A resposta
deve manter Minimal API, acesso assíncrono e declarar o status produzido. A
instruction de testes não deve ser carregada só por essa alteração.

### 3. Escopo dos testes

Com `src/Tests/Api.Tests/TrainingCreationTests.cs` aberto, envie:

```text
Adicione um teste funcional para comprovar que criar um treinamento com
durationHours igual a zero retorna HTTP 400 e o erro esperado.
```

Observe `GERAL:`, `TESTES:` e a referência a `tests.instructions.md`. O teste
deve usar `TrainingCatalogApiFactory` e `HttpClient`, começar com `Returns` e
verificar o status antes do corpo. A instruction da API não deve ser necessária
se nenhum arquivo de `src/Api` for alterado.

### 4. Combinação dos dois escopos

Envie:

```text
Implemente na API um endpoint GET /api/trainings/count e crie um teste
funcional para ele.
```

Como a tarefa envolve `src/Api` e `src/Tests`, procure as duas instructions
específicas nas referências e os marcadores `API:` e `TESTES:`.

### 5. Anexo manual

No seletor de contexto do chat, use **Add Context > Instructions** e anexe
`API do Training Catalog`. Com `src/Application/Training.cs` aberto, envie:

```text
Sem alterar arquivos, avalie este contrato como se ele fosse usado por um
novo endpoint da API.
```

Mesmo fora do glob `src/Api/**/*.cs`, a referência e o marcador `API:` devem
aparecer porque a instruction foi anexada explicitamente. Remova o anexo e
repita em uma nova conversa para comparar.

### 6. `AGENTS.md`

Envie:

```text
Para adicionar uma propriedade ao modelo, quais arquivos deste repositório
você evitaria editar sem que uma mudança de esquema tivesse sido solicitada?
Não altere arquivos.
```

A resposta deve identificar migrations e `src/Api/training-catalog.db`, regras
que existem somente em `AGENTS.md`.

> [!TIP]
> Depois de cada prompt que altera código, descarte as mudanças antes de
> executar o próximo cenário. Isso mantém cada evidência independente.

## Referências

- [Demonstração 1 — Escopo correto, contexto menor](https://github.com/impacta-ghcp-eng-moderna/material/blob/main/modulo-02/plano-modulo-02.md#demonstra%C3%A7%C3%A3o-1--escopo-correto-contexto-menor)
- [Custom instructions no VS Code](https://code.visualstudio.com/docs/agent-customization/custom-instructions)
- [Imagem Dev Container para .NET](https://mcr.microsoft.com/en-us/artifact/mar/devcontainers/dotnet/about)
