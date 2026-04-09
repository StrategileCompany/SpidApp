# CLAUDE.md

## Projeto
- Nome: `xpid`
- Domínio: gestão comercial multi-tenant para operação de vendas, compras, estoque, recebíveis e financeiro
- Superfícies: web (`apps/web`), mobile (`apps/mobile`) e backend legado Fastify (`apps/api`)

## Stack real
- Node: `>=20`
- pnpm: `>=9`
- TypeScript: `5.7.3`
- Prisma / `@prisma/client`: `6.4.1`
- Zod: `3.24.2`
- Web: Next.js `15.2.0`, React `19.0.0`, React DOM `19.0.0`, `jose` `6.1.3`
- Mobile: Vite `6.1.0`, React Router DOM `7.2.0`, Capacitor core `7.1.0`, Capacitor Android `7.5.0`, Capacitor Preferences `7.0.0`
- API legada: Fastify `5.2.1`, `@fastify/cors` `10.0.2`, `tsx` `4.19.3`
- Android: `minSdk 23`, `compileSdk 35`, `targetSdk 35`, `applicationId com.xpid.app`

## Estrutura de pastas
- `apps/web`: frontend principal e API Next em `src/app/api`
- `apps/mobile`: app React + Vite + shell Android via Capacitor
- `apps/api`: servidor Fastify legado/paralelo
- `packages/shared`: schemas Zod, tipos e utilitários de domínio
- `packages/ui`: componentes React compartilhados
- `prisma`: schema e migrations
- `.docs`: documentação operacional para agentes
- `.github`: automações/versionamento do repositório
- `.husky`: hooks de git

## Scripts da raiz
- `pnpm dev`: roda workspaces em paralelo
- `pnpm dev:api`: sobe `@xpid/api`
- `pnpm dev:web`: sobe `@xpid/web`
- `pnpm dev:mobile`: sobe `@xpid/mobile`
- `pnpm build`: build recursivo
- `pnpm lint`: ESLint no monorepo
- `pnpm format`: Prettier
- `pnpm test`: tenta executar `test` nos workspaces
- `pnpm db:migrate`: `prisma migrate dev` via `@xpid/api`
- `pnpm db:generate`: gera Prisma client
- `pnpm db:studio`: abre Prisma Studio

## Scripts por app
- `apps/web`: `dev`, `build`, `start`, `postinstall`
- `apps/mobile`: `dev`, `build`, `sync`
- `apps/api`: `dev`, `build`, `start`
- `packages/shared`: `lint`, `typecheck`
- `packages/ui`: `lint`, `typecheck`

## Banco e domínio
- Prisma usa PostgreSQL via `DATABASE_URL`
- Schema atual tem 26 modelos
- Núcleos: tenancy, catálogo, mestres, compras, estoque, vendas, recebíveis, pagamentos/financeiro, settings e sync
- Todo dado de negócio do web é filtrado por `tenantId`

## Regras de negócio críticas
- Registro cria tenant, tabela de preço padrão, contas padrão e `app_settings`
- Venda `DRAFT` não deveria consolidar estoque/financeiro; `CONFIRMED` gera cupom, baixa estoque e pagamentos/recebíveis
- Pagamento imediato (`CASH`, `PIX`, `DEBIT_CARD`, cartão sem parcelamento) vira `payment` + `finance_entry` pago
- Crediário/boleto/cheque e cartão parcelado geram `receivables`
- Estoque é orientado a movimentos e usa unidade base por fator de conversão
- Método de custo exposto em settings: `FIFO` ou `AVERAGE`; o código de confirmação de venda implementa FIFO
- Soft-delete existe para `Product`, `Customer` e `Supplier` via `deletedAt`

## Autenticação e acesso
- Web usa email/senha com hash `scrypt`
- Sessão via cookie `xpid_token`
- Mobile usa Bearer token salvo localmente
- `apps/web/src/middleware.ts` protege páginas e APIs; públicos: `/login`, `/registro`, `/api/auth/*`, `/api/health`

## Padrões e convenções
- Código em inglês; UI e docs em PT-BR
- App Router no web; páginas em `apps/web/src/app`
- Componentes compartilhados preferencialmente em `packages/ui`
- Schemas de entrada centralizados em `packages/shared/src/schemas`
- Formatação via Prettier e lint via ESLint
- `next.config.js` transpila `@xpid/shared` e `@xpid/ui`
- `lint-staged` aplica `eslint --fix` e `prettier --write`

## Deploy e ambientes
- Web: Vercel, configurado por `vercel.json`
- Build do deploy: `cd apps/web && npx prisma generate --schema=../../prisma/schema.prisma && next build`
- Mobile: build local Vite + Capacitor Android; há APKs versionados na raiz
- Não há URL de produção fixa garantida por config, mas o mobile sugere `/api` no mesmo host ou `https://xpid.vercel.app/api` como valor operacional

## Variáveis relevantes
- `DATABASE_URL`: obrigatória para Prisma
- `JWT_SECRET`: usada no web; sem valor cai em fallback inseguro
- `VITE_API_URL`: base opcional da API no mobile
- `PORT`: usada apenas em `apps/api`
- `SYNC_SECRET`: usada apenas em `apps/api` e no fluxo legado mobile
- `.env.example` ainda lista `APP_INSTANCE_ID` e `NEXT_PUBLIC_API_URL`, mas o web atual não depende delas

## Áreas de atenção antes de mexer
- `apps/api` e `apps/web/src/app/api` coexistem; trate `apps/api` como legado até decisão explícita
- `apps/mobile/src/lib/database.ts` ainda é stub/no-op em browser dev
- `apps/mobile/src/lib/sync.ts` não está alinhado com o contrato das rotas Next de sync
- `apps/web/src/app/configuracoes/page.tsx` consome `/settings/printers`, mas não existe rota correspondente
- `apps/web/src/app/clientes/[id]/page.tsx` tem `TODO` para receber parcela no detalhe
