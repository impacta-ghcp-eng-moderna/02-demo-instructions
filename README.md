# Demo: escopo correto, contexto menor

Este repositório prepara a demonstração de **custom instructions** do Módulo 2.
Ele usa a aplicação Training Catalog do
[`01-lab`](https://github.com/impacta-ghcp-eng-moderna/01-lab/tree/main/src)
para mostrar que regras gerais e regras específicas podem ser combinadas sem
repetir todo o contexto em cada prompt.

## Como os arquivos de instructions funcionam

Antes de comparar os arquivos, separe dois conceitos:

- **escopo de armazenamento** define onde a instruction fica e com quem ela é
  compartilhada;
- **escopo de aplicação** define em quais tarefas ou arquivos ela entra
  automaticamente no contexto.

Na janela **Agent Customizations**, **New Instructions (Workspace)** e
**New Instructions (User)** escolhem o escopo de armazenamento, não o alcance
automático da regra:

| Opção | Local padrão | Compartilhamento | Aplicação |
| --- | --- | --- | --- |
| **Workspace** | `.github/instructions/*.instructions.md` | Versionado com o repositório | Conforme `applyTo` ou anexo manual |
| **User** | `~/.copilot/instructions/*.instructions.md` | Pessoal, disponível entre workspaces | Conforme `applyTo` ou anexo manual |
| **Repository-wide** | `.github/copilot-instructions.md` | Versionado com o repositório | Sempre ativo no workspace |
| **Agent instructions** | `AGENTS.md` | Versionado e interoperável | Ativo no workspace ou conforme sua localização aninhada |

Assim, `api.instructions.md` e `tests.instructions.md` são **Workspace
Instructions**, enquanto `noir.instructions.md`, criado mais adiante, é uma
**User Instruction**. Uma Workspace Instruction não se aplica necessariamente
a todo o workspace: seu front matter `applyTo` controla a aplicação
automática. Sem `applyTo`, o arquivo ainda pode ser anexado manualmente.

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

Não existe uma lista universal como “pessoal vence repositório” ou
“`AGENTS.md` vence `copilot-instructions.md`”. Pense no processo em três
etapas:

1. **Descoberta:** a ferramenta localiza as instructions disponíveis no
   perfil, no workspace e, quando suportado, em diretórios do repositório.
2. **Aplicabilidade:** as instructions gerais são incluídas automaticamente;
   um arquivo `*.instructions.md` entra quando seu `applyTo` corresponde aos
   arquivos da tarefa ou quando é anexado manualmente.
3. **Combinação:** todas as instructions aplicáveis são adicionadas ao
   contexto. A ordem entre formatos não é garantida, portanto uma regra não
   deve depender de “sobrescrever” outra.

Neste repositório, `.github/copilot-instructions.md` e o `AGENTS.md` da raiz
são gerais. `api.instructions.md` entra ao trabalhar em arquivos
`src/Api/**/*.cs`, enquanto `tests.instructions.md` entra para
`src/Tests/**/*.cs`. Se houver vários `AGENTS.md` aninhados, o mais próximo do
arquivo em que o agente está trabalhando tem precedência entre esses
`AGENTS.md`; isso não cria uma precedência geral sobre os outros formatos.

Se duas instructions aplicáveis se contradisserem, o resultado pode variar.
Prefira regras gerais na raiz e regras complementares, mais específicas, nos
escopos por caminho. Use as referências da resposta — ou `/instructions` no
Copilot CLI — para confirmar quais arquivos foram realmente considerados.

As custom instructions afetam o chat e os agentes, mas não as sugestões
inline mostradas enquanto se digita no editor.

## Monorepos e workspaces abertos em subpastas

A pasta aberta como workspace influencia a descoberta das customizações. Este
repositório assume que sua raiz foi aberta; assim, os globs como
`src/Api/**/*.cs` são avaliados a partir da estrutura esperada.

### VS Code

Por padrão, ao abrir somente uma subpasta de um monorepo, o VS Code pode não
descobrir as customizações que estão acima da raiz desse workspace. Para
incluir as customizações do repositório pai:

1. Abra **Settings**.
2. Procure por `chat.useCustomizationsInParentRepositories`.
3. Habilite **Chat: Use Customizations In Parent Repositories**.
4. Inicie uma nova conversa e confira as referências carregadas.

Em `settings.json`, a configuração equivalente é:

```json
{
  "chat.useCustomizationsInParentRepositories": true
}
```

Mesmo com essa opção, mantenha os padrões `applyTo` coerentes com a raiz usada
para organizar o monorepo. Para uma demonstração previsível deste repositório,
abra sua raiz, e não somente `src`.

### Copilot CLI

Ao ser iniciado em uma subpasta, o Copilot CLI procura instructions no
diretório de trabalho, nos diretórios intermediários e na raiz do repositório.
Assim, iniciar o CLI em `src/Api` ainda permite descobrir
`.github/copilot-instructions.md` e `AGENTS.md` da raiz. Instructions modulares
continuam sendo incluídas somente quando `applyTo` corresponde a um arquivo em
que o CLI está trabalhando.

Execute `/instructions` para visualizar, habilitar ou desabilitar os arquivos
descobertos na sessão. Depois de editar uma instruction, use `/new` ou reinicie
a sessão do CLI para carregar a nova versão.

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

## Demonstrar uma User Instruction no Codespace

Uma User Instruction pertence ao perfil do usuário no ambiente, não ao
repositório. Por isso, ela pode ser combinada com as instructions deste
workspace sem ser adicionada ao Git.

Para criar uma instruction pessoal com efeito imediatamente visível:

1. No Codespace, abra a Command Palette com `Ctrl+Shift+P`.
2. Execute **Chat: Open Customizations**.
3. Abra a aba **Instructions**.
4. No menu de criação, selecione **New Instructions (User)**.
5. Informe o nome `noir`. O arquivo será apresentado como
   `noir.instructions.md`.
6. Substitua o conteúdo pelo texto abaixo e salve:

```markdown
---
name: Narrador noir
description: Torna o estilo das respostas imediatamente reconhecível.
applyTo: "**"
---

- Comece todas as respostas com `DETETIVE:`.
- Escreva em tom dramático de filme noir, usando frases curtas.
- Não altere nomes, código ou conteúdo técnico por causa do estilo.
- Termine todas as respostas com `Caso encerrado.`
```

7. Inicie uma nova conversa e envie:

```text
Explique brevemente a responsabilidade do projeto Application.
```

A resposta deve começar com `DETETIVE:`, usar o tom noir e terminar com
`Caso encerrado.`. Nas referências ou customizações carregadas, localize
`Narrador noir` com origem de usuário e compare-a com as instructions de
origem workspace.

Como `applyTo` vale `**`, a regra é automática para qualquer arquivo. Depois
da demonstração, desabilite ou exclua `Narrador noir` no mesmo editor para que
ela não afete os próximos prompts. Para reutilizar User Instructions em outros
ambientes, habilite **Settings Sync** e inclua **Prompts and Instructions**.

## Referências

- [Demonstração 1 — Escopo correto, contexto menor](https://github.com/impacta-ghcp-eng-moderna/material/blob/main/modulo-02/plano-modulo-02.md#demonstra%C3%A7%C3%A3o-1--escopo-correto-contexto-menor)
- [Custom instructions no VS Code](https://code.visualstudio.com/docs/agent-customization/custom-instructions)
- [Custom instructions no Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Imagem Dev Container para .NET](https://mcr.microsoft.com/en-us/artifact/mar/devcontainers/dotnet/about)
