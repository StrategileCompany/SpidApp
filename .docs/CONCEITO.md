# CONCEITO — Xpid

## Leitura operacional do produto
Xpid é um sistema de operação comercial para pequenos vendedores e distribuidores que precisam vender, comprar, controlar estoque, cobrar clientes e fechar caixa em web e mobile. O modelo dominante no código é B2B SaaS multi-tenant: cada empresa é um `Tenant`, com isolamento completo de catálogo, clientes, fornecedores, vendas, financeiro e configurações.

## Modelo de conta e tenancy
- Cada conta de empresa nasce em `POST /api/auth/register` com `email`, `password` e `companyName`.
- O cadastro inicial já cria um tenant, uma tabela de preço padrão, contas financeiras padrão (`Carteira` e `Banco`) e `app_settings`.
- Toda leitura e escrita do app web é filtrada por `tenantId`.
- O mobile atual usa token JWT em `Authorization: Bearer`, mas ainda preserva artefatos legados de `APP_INSTANCE_ID` e `SYNC_SECRET` em armazenamento local.

## Domínios de negócio centrais

### Catálogo
- Produto pertence a categoria e pode pertencer a subcategoria.
- Produto possui múltiplas unidades (`product_units`) com `factor_to_base`.
- Preço é definido por combinação produto + unidade + tabela de preço.
- Estoque mínimo existe por produto (`minStock`) para alertas operacionais.

### Compras e custo
- Compra confirmada gera itens, custos extras e pagamentos.
- `cost_lots` indica rastreamento de custo por lote.
- O app expõe escolha de método de custo (`FIFO` ou `AVERAGE`) nas configurações.
- A confirmação de venda já consome FIFO em `apps/web/src/lib/sales-logic.ts`; não existe implementação equivalente de custo médio no backend atual.

### Estoque
- Estoque é orientado a eventos via `inventory_movements`.
- As quantidades operacionais são convertidas para unidade base.
- Inventário/contagem gera ajuste por diferença, não edição direta de saldo.
- Venda confirmada gera saída `OUT`; compra e contagem geram entradas/ajustes por rotas específicas.

### Vendas
- Venda suporta `DRAFT`, `CONFIRMED` e `CANCELLED`.
- Vendas confirmadas geram número de cupom sequencial e movimentação de estoque.
- O formulário web de nova venda confirma imediatamente por padrão.
- Split de pagamento é nativo e aceita dinheiro, PIX, cartões, crediário, boleto e cheque.

### Recebíveis e inadimplência
- Recebíveis existem para crediário, boleto, cheque e cartão parcelado.
- Pagamento imediato gera `payment` e `finance_entry` quitado.
- Parcelamento gera `receivables` em aberto com vencimentos calculados.
- O produto trata inadimplência como fluxo principal: modal no mobile, tela dedicada no web e liquidação manual por recebível.

### Financeiro
- Há contas financeiras com tipo (`CASH`, `BANK`, `OTHER`) e métodos de pagamento padrão.
- Entradas financeiras possuem ciclo `SCHEDULED | DUE | PAID | CANCELLED`.
- Fechamento mensal é por conta e mês (`monthly_closures`).
- O dashboard calcula faturamento, lucro bruto, lucro líquido, recebido e a receber.

### Configurações
- `app_settings.data` concentra dados do cupom, método de custo, formato padrão de impressão e tabela de preço padrão.
- Web possui área administrativa para dados da empresa, método de custo, contas, tabelas de preço, impressão e sync.
- Mobile possui página própria de configurações, mas ainda baseada em `localStorage` e chaves legadas.

## Offline e sincronização
- O desenho de produto continua offline-first: `outbox_operations`, `sync_cursors` e cliente mobile de push/pull existem no código.
- O SQLite local do mobile ainda está em estado parcial: a implementação corrente é um stub/no-op para browser dev em `apps/mobile/src/lib/database.ts`.
- O contrato de sync do mobile não está alinhado com as rotas Next atuais:
  - mobile espera `accepted/rejected` no push e `changes/hasMore` no pull;
  - web responde `applied/skipped/errors` no push e coleções planas por entidade no pull.
- Existe também um backend Fastify legado em `apps/api` com rotas de sync próprias.

## Monetização deduzida do código
- O código não implementa cobrança, assinatura ou faturamento do próprio produto.
- A presença de `Tenant`, autenticação por empresa e isolamento por `tenantId` indica produto preparado para SaaS multi-tenant.
- Como não há módulo de billing, qualquer monetização atual está fora deste repositório.
