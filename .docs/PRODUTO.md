# PRODUTO — Xpid

## Superfícies ativas
- Web: App Router em `apps/web/src/app`
- Mobile: SPA React Router em `apps/mobile/src`
- API: rotas Next em `apps/web/src/app/api`
- Shared UI: `packages/ui`

## Checklist funcional por área

### Acesso e tenancy
- [x] Registro de empresa com criação automática de tenant, contas padrão e settings
- [x] Login por email/senha com JWT
- [x] Proteção de páginas e APIs por middleware

### Dashboard e relatórios
- [x] Dashboard web com faturamento, lucro bruto, lucro líquido, ticket médio, recebido e a receber
- [x] Lista de vendas recentes no dashboard
- [x] Relatórios API `dashboard`, `products-3m`, `customers-3m`, `cashflow`

### Produtos e catálogo
- [x] CRUD de produtos no web
- [x] CRUD de produtos no mobile
- [x] Categorias e subcategorias no web
- [x] Unidades vendáveis por fator de conversão
- [x] Tabelas de preço por produto/unidade/tier

### Clientes e inadimplência
- [x] CRUD de clientes no web
- [x] CRUD de clientes no mobile
- [x] Tela de inadimplentes no web com liquidação manual
- [x] Modal de inadimplência na abertura do mobile
- [ ] Ação de receber diretamente a partir do detalhe de cliente está pendente no web (`TODO` no código)

### Fornecedores e compras
- [x] CRUD de fornecedores no web
- [x] CRUD de fornecedores no mobile
- [x] Cadastro de compra com itens, custos extras e pagamentos
- [x] Detalhe/listagem de compras nas duas superfícies

### Vendas
- [x] Nova venda web com fluxo de cliente, itens, desconto/frete e split de pagamento
- [x] Nova venda mobile
- [x] Confirmação de venda com cupom sequencial
- [x] Detalhe de venda no web
- [x] Cancelamento de venda no web e no mobile
- [x] Suporte a crediário, boleto, cheque e cartão parcelado
- [ ] Recebimento de parcela no detalhe da venda mobile ainda não está conectado a ação de API

### Estoque
- [x] Visão geral de estoque no web e no mobile
- [x] Contagem/inventário no web e no mobile
- [x] Lista de movimentações
- [x] Consulta de estoque por produto

### Financeiro
- [x] Visão geral financeira no web e no mobile
- [x] Gestão de contas
- [x] Lançamentos financeiros
- [x] Fechamento mensal
- [x] Categorias financeiras

### Configurações e impressão
- [x] Dados do cupom no web
- [x] Método de custo configurável no web e no mobile
- [x] Formato padrão de impressão no web e no mobile
- [x] Gestão de tabelas de preço e contas no web
- [ ] Perfis de impressora no web dependem de endpoint ausente (`/settings/printers`)
- [x] Configurações locais de impressora no mobile

### Offline e sync
- [x] Outbox e cursor modelados no schema Prisma
- [x] Cliente mobile com engine de push/pull e auto-sync
- [x] Rotas de sync no Next e no Fastify legado
- [ ] Persistência local efetiva do mobile ainda não está implementada no stub atual
- [ ] Contrato de sync mobile x Next está desalinhado

## Componentes e blocos reutilizáveis
- [x] `PaymentSplit`
- [x] `InstallmentConfig`
- [x] `CouponPreview`
- [x] `OverdueAlert` / `OverdueModal`
- [x] `Layout` / `MobileLayout`
- [x] `DataTable`
- [x] `StatsCard`
- [x] componentes base de `@xpid/ui` (`Button`, `Input`, `Modal`, `Table`, `Toast`, etc.)
