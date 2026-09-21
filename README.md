# CobrancaApi

API de gestão de débitos, parcelamento e pagamentos — projeto de portfólio em **C# / ASP.NET Core 8** com **Entity Framework Core**, seguindo Clean Architecture, totalmente em português.

O domínio é genérico (débito → parcelamento → pagamento, com cálculo de multa e juros de mora), inspirado em experiência real com sistemas de cobrança/tributário — sem reproduzir nenhuma estrutura ou regra de um sistema real específico.

## Arquitetura

```
src/
  CobrancaApi.Dominio/         entidades e regras de negócio puras (sem dependências externas)
  CobrancaApi.Aplicacao/       casos de uso, DTOs, interfaces (serviços e repositórios)
  CobrancaApi.Infraestrutura/  EF Core: DbContext e implementação dos repositórios
  CobrancaApi.Api/             controllers, Program.cs, injeção de dependência, Swagger
tests/
  CobrancaApi.Testes/          testes unitários (xUnit)
```

A regra: **Dominio** não depende de nada. **Aplicacao** depende só de **Dominio**. **Infraestrutura** implementa as interfaces definidas em **Aplicacao**. **Api** conecta tudo via injeção de dependência.

Nomes de domínio (entidades, propriedades, métodos) estão em português. Termos técnicos universais — `Controller`, `DbContext`, `Request`/`Response`, `DTO` — ficam em inglês, como é convenção mesmo em times 100% brasileiros.

## Regras de negócio implementadas

- Um débito (`Debito`) tem valor original, vencimento, multa (%) e juros de mora diário (%).
- `Debito.CalcularValorAtualizado()` calcula o valor atualizado pró-rata die quando vencido.
- Um débito pode ser parcelado em N parcelas, dividindo o valor **já atualizado**.
- Ao registrar o pagamento da última parcela pendente, o débito é quitado automaticamente.

## Como rodar localmente

1. Instale o [.NET 8 SDK](https://dotnet.microsoft.com/download).
2. Por padrão o projeto já roda com banco em memória (`UseInMemoryDatabase`) — não precisa instalar SQL Server pra testar. Quando quiser persistência de verdade, troque pela linha `UseSqlServer` (comentada) no `Program.cs` e ajuste a connection string em `appsettings.json`.
3. Na raiz do projeto:
   ```bash
   dotnet restore
   dotnet build
   dotnet run --project src/CobrancaApi.Api
   ```
4. Abra `https://localhost:<porta>/swagger` pra testar os endpoints.

Se o `dotnet restore` reclamar de não achar pacotes (comum em rede corporativa restrita), veja o `NuGet.Config` na raiz — ele já aponta pro nuget.org sem precisar mexer na config global da máquina.

Se quiser gerar as migrations do EF Core:
```bash
dotnet ef migrations add InitialCreate --project src/CobrancaApi.Infraestrutura --startup-project src/CobrancaApi.Api
dotnet ef database update --project src/CobrancaApi.Infraestrutura --startup-project src/CobrancaApi.Api
```

Para rodar os testes:
```bash
dotnet test
```

## Endpoints principais

| Método | Rota                                   | Descrição                                  |
|--------|-----------------------------------------|---------------------------------------------|
| GET    | `/api/debitos`                          | Lista débitos com valor atualizado          |
| POST   | `/api/debitos`                          | Cria um débito                              |
| POST   | `/api/debitos/{debitoId}/parcelamento`  | Gera parcelamento para um débito            |
| POST   | `/api/pagamentos`                       | Registra pagamento de uma parcela           |

## Roteiro de evolução (próximos passos sugeridos)

- [ ] Adicionar `DevedoresController` (CRUD de devedores)
- [ ] Autenticação JWT
- [ ] Migrations reais + seed de dados de exemplo
- [ ] Testes de integração com `WebApplicationFactory`
- [ ] Paginação e filtros no `GET /api/debitos`
- [ ] Front-end simples (Angular ou React) consumindo a API
- [ ] Deploy (Azure App Service / Render / Railway) pra ter link ao vivo no currículo

## Por que este projeto no portfólio

Mostra, num escopo pequeno: modelagem de domínio, separação de camadas, regra de negócio não-trivial (cálculo financeiro com data), EF Core com relacionamentos, testes unitários da regra mais importante. É pensado pra sustentar uma conversa de entrevista sobre *por que* cada decisão foi tomada — não só "o que" foi feito.
