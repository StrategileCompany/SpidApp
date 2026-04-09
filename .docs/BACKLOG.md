# BACKLOG — Xpid
Próximo ID: T-005

## Histórico
- 2026-02-21: Especificacao tecnica completa (v1.0)

## Sprint Atual
Ver `tasklist.md` para checklist detalhado.
Prioridade historica registrada: Fundacao (monorepo + config) → Offline-first → Produtos → Compras → Vendas → Financeiro

## Em análise
- T-001 — Alinhar contrato de sincronização entre mobile e Next API
  - Contexto: `apps/mobile/src/lib/sync.ts` espera `accepted/rejected` no push e `changes/hasMore` no pull, mas `apps/web/src/app/api/sync/*` responde outro formato.
  - Impacto: sync offline não fecha ciclo fim a fim com o backend Next atual.
  - Áreas: `apps/mobile/src/lib/sync.ts`, `apps/web/src/app/api/sync/push/route.ts`, `apps/web/src/app/api/sync/pull/route.ts`

- T-002 — Implementar persistência local real para o mobile
  - Contexto: `apps/mobile/src/lib/database.ts` inicializa schema, mas o driver atual é stub/no-op para browser dev e não executa persistência real.
  - Impacto: o modo offline não pode ser validado integralmente a partir da implementação atual.
  - Áreas: `apps/mobile/src/lib/database.ts`, integração Capacitor SQLite

- T-003 — Resolver consumo de `/settings/printers` no web
  - Contexto: `apps/web/src/app/configuracoes/page.tsx` usa `useApi('/settings/printers')`, porém não existe rota correspondente em `apps/web/src/app/api`.
  - Impacto: seção de impressoras do painel web opera com lacuna de dados.
  - Áreas: `apps/web/src/app/configuracoes/page.tsx`, `apps/web/src/app/api/settings/*`

- T-004 — Fechar ações de recebimento pendentes nas telas de detalhe
  - Contexto: há `TODO` em detalhe de cliente no web e botão visual de `Receber` no detalhe de venda mobile sem fluxo final de liquidação.
  - Impacto: usuário precisa sair do contexto principal para liquidar alguns recebíveis.
  - Áreas: `apps/web/src/app/clientes/[id]/page.tsx`, `apps/mobile/src/pages/sales/SaleDetail.tsx`

## Backlog Futuro
- Exportacao CSV/PDF de relatorios
- Alertas Android agendados para vencimentos
- Modo "Catalogo rapido" com favoritos
- Impressao com logo e QR

## Nao-Escopo
- NFC-e / nota fiscal
- Integracao bancaria automatica
- Importacao de planilhas
- WhatsApp API (apenas compartilhar PDF via share sheet)
