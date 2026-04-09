# TECNICO — Xpid

## Monorepo e versões reais
- Workspace: `pnpm-workspace.yaml`
- Runtime raiz: Node `>=20`, pnpm `>=9`
- TypeScript: `5.7.3`
- ESLint: `9.20.0`
- Prettier: `3.5.2`
- Prisma CLI/client: `6.4.1`
- Zod: `3.24.2`

### apps/web
- Next.js `15.2.0`
- React / React DOM `19.0.0`
- `jose` `6.1.3`
- Build: `next build`
- Observações:
  - `vercel.json` aponta o deploy para `apps/web`
  - `next.config.js` transpila `@xpid/shared` e `@xpid/ui`
  - `eslint.ignoreDuringBuilds = true`

### apps/mobile
- Vite `6.1.0`
- React / React DOM `19.0.0`
- React Router DOM `7.2.0`
- Capacitor core `7.1.0`
- Capacitor Android `7.5.0`
- Capacitor Preferences `7.0.0`
- Android:
  - `applicationId`: `com.xpid.app`
  - `minSdkVersion`: `23`
  - `compileSdkVersion`: `35`
  - `targetSdkVersion`: `35`

### apps/api
- Fastify `5.2.1`
- `@fastify/cors` `10.0.2`
- `tsx` `4.19.3`
- Papel atual: backend legado/paralelo ao backend do Next

### packages
- `@xpid/shared`: schemas Zod, tipos e utilitários de domínio
- `@xpid/ui`: biblioteca de componentes React reutilizáveis

## Infraestrutura deduzida do código
- Banco principal: PostgreSQL via Prisma (`datasource db provider = "postgresql"`)
- Deploy web: Vercel (`vercel.json`)
- Aplicação mobile: bundle Vite + shell Android via Capacitor
- Sem Docker, CI ou IaC versionados no repositório
- Há `.github/` e `.husky/`, mas sem pipeline de infraestrutura explícito nos arquivos inspecionados

## Ambientes e variáveis

### Consumidas de fato
- `DATABASE_URL`
  - obrigatória para Prisma e para `apps/api`
- `JWT_SECRET`
  - usada no web para JWT; se ausente cai em fallback inseguro hardcoded
- `VITE_API_URL`
  - lida no mobile para resolver a base da API
- `PORT`
  - usada apenas em `apps/api`
- `SYNC_SECRET`
  - usada apenas no backend Fastify legado

### Presentes em `.env.example`, mas não centrais no fluxo web atual
- `APP_INSTANCE_ID`
- `NEXT_PUBLIC_API_URL`

## Arquitetura de aplicação

### Backend efetivo no repositório
- Web e API convivem em `apps/web`
- Rotas Next App Router em `apps/web/src/app/api`
- Namespaces disponíveis:
  - `auth`
  - `categories` e `subcategories`
  - `customers`
  - `finance`
  - `health`
  - `inventory`
  - `price-tiers`
  - `products`
  - `purchases`
  - `receivables`
  - `reports`
  - `sales`
  - `settings`
  - `suppliers`
  - `sync`

### Backend legado
- `apps/api` ainda sobe um servidor Fastify com rotas equivalentes para:
  - `health`
  - `categories`
  - `products`
  - `customers`
  - `suppliers`
  - `purchases`
  - `sales`
  - `inventory`
  - `receivables`
  - `finance`
  - `sync`
  - `reports`

## Autenticação
- Web/Next usa cadastro e login por email/senha
- Hash de senha: `crypto.scryptSync`
- Sessão:
  - cookie `xpid_token` para web
  - Bearer token para mobile
- Middleware do Next protege páginas e APIs, exceto:
  - `/login`
  - `/registro`
  - `/api/auth/*`
  - `/api/health`

## Schema Prisma
- Total de modelos no schema atual: `26`
- Núcleos:
  - tenancy: `Tenant`
  - catálogo: `Category`, `Subcategory`, `Product`, `ProductUnit`, `PriceTier`, `ProductPrice`
  - mestres: `Customer`, `Supplier`
  - compras: `Purchase`, `PurchaseItem`, `PurchaseCost`
  - estoque: `InventoryMovement`, `CostLot`
  - vendas: `Sale`, `SaleItem`
  - recebíveis: `Receivable`, `ReceivableSettlement`
  - pagamentos/financeiro: `Payment`, `Account`, `FinanceCategory`, `FinanceEntry`, `MonthlyClosure`
  - configuração/sync: `AppSettings`, `OutboxOperation`, `SyncCursor`

## Inconsistências técnicas atuais
- `apps/mobile/src/lib/database.ts` não persiste SQLite real no ambiente web; hoje opera como stub.
- `apps/mobile/src/lib/sync.ts` não casa com o contrato das rotas Next de sync.
- `apps/web/src/app/configuracoes/page.tsx` consome `/settings/printers`, mas não existe rota correspondente em `apps/web/src/app/api`.
- O repositório mantém duas superfícies de backend ativas (`apps/web` e `apps/api`), com risco de documentação e env vars divergentes.
