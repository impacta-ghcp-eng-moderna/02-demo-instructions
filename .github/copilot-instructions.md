# Instruções gerais do Training Catalog

- Considere `src/TrainingCatalog.slnx` a solução principal e mantenha a separação entre `Api`, `Application`, `Infrastructure`, `Client` e `Tests`.
- Use C# com nullable reference types habilitados e recursos compatíveis com .NET 10.
- Preserve os termos de domínio em inglês no código e as mensagens apresentadas ao usuário em português do Brasil.
- Faça somente as alterações necessárias para a solicitação e valide-as com o menor comando `dotnet` que cubra o comportamento alterado.
- Ao responder a uma solicitação de alteração, comece o resumo da solução com `GERAL:` para tornar visível a aplicação destas instruções.
