## podium-web-corporate

> Portal B2B para corporações gerenciarem funcionários, centros de custo e faturamento no ecossistema Podium. React + TypeScript + Vite + Chakra UI + TanStack Query.

# Copilot Instructions - Podium Web Corporate

## Visão Geral

Portal B2B para corporações gerenciarem funcionários, centros de custo e faturamento no ecossistema Podium. React + TypeScript + Vite + Chakra UI + TanStack Query.

## Arquitetura

### Stack Principal
- **Frontend**: React 18.3 + TypeScript (strict mode)
- **Build**: Vite 5.4 rodando em `localhost:5175`
- **UI**: Chakra UI com tema customizado Podium (midnight + gold)
- **State**: TanStack Query para server state, Context API para auth
- **Routing**: React Router DOM v7 com PrivateRoute wrapper

### Estrutura de Camadas
```
App.tsx (ChakraProvider → AuthProvider → BrowserRouter)
  └─ PrivateRoute (autenticação)
      └─ MainLayout (layout comum)
          └─ Pages (Dashboard, Employees, CostCenters, Billing)
```

## Convenções Críticas

### 1. API & Autenticação
- Base URL via `VITE_API_URL` (fallback: `http://localhost:8000`)
- Auth usa `x-www-form-urlencoded` para login (campo `username` recebe email)
- Token em `@Podium:token`, user em `@Podium:user` (localStorage)
- Interceptor global em [src/services/api.ts](src/services/api.ts) adiciona `Authorization: Bearer`

**Exemplo de integração**:
```typescript
// Novo endpoint sempre em src/services/api.ts
export interface NewResource { id: number; name: string }
// Hook sempre com TanStack Query ou useState + useEffect (ver useDashboard.ts)
```

### 2. Tema Podium (Identidade Visual)
- Cores: `midnight.900` (bg escuro), `gold.600` (primária dourada)
- Fonts: Montserrat (headings), Inter (body)
- Cards: usar `layerStyle="card"` com hover gold glow
- Inputs: `_focus` sempre com `borderColor: "gold.600"`

**Ver**: [src/theme/index.ts](src/theme/index.ts) para paleta completa e layer styles

### 3. Componentes Reutilizáveis
- **Barrel Exports**: sempre exportar via `index.ts` (ex: `components/UI/index.ts`)
- **FormInput/FormSelect**: customizações de Chakra com validação integrada
- **StatCard**: dashboards seguem padrão de KPI cards com cores temáticas

**Exemplo**:
```tsx
import { FormInput, StatCard } from "../components";
// NÃO: import FormInput from "../components/UI/FormInput";
```

### 4. Data Fetching Pattern
- **Hooks customizados** para lógica de API (ex: `useDashboard.ts`)
- Sempre retornar `{ data, loading, error }` trio
- Fallback para dados mockados em `catch` durante dev
- Preferir TanStack Query para cache e revalidation

### 5. TypeScript
- `tsconfig.json` herda de `../tsconfig.base.json` (monorepo)
- Strict mode ativado (`noUnusedLocals`, `noUnusedParameters`)
- Interfaces de API sempre em [src/services/api.ts](src/services/api.ts)

## Workflows de Desenvolvimento

### Comandos
```bash
yarn dev         # Vite dev server em 0.0.0.0:5175
yarn build       # Build de produção
yarn lint        # ESLint com max-warnings 0
yarn format:check  # Prettier check
```

### Criar Nova Página
1. Criar componente em `src/pages/`
2. Adicionar rota em [src/App.tsx](src/App.tsx) dentro de `<PrivateRoute>` e `<MainLayout>`
3. Se precisar de API: adicionar tipos/interfaces em [api.ts](src/services/api.ts)
4. Criar hook customizado em `src/hooks/` se houver lógica complexa

### Adicionar Componente Reutilizável
1. Criar em `src/components/{Cards|Tables|UI}/`
2. Exportar via barrel em `index.ts` local e global
3. Seguir padrão de props com Chakra `ComponentProps` extension

## Pontos de Integração

### Backend API (FastAPI esperado)
- Endpoints corporativos prefixados com `/api/v1/stats/corporate/`
- Login: `POST /api/v1/auth/login` (form-urlencoded) — **atenção: prefixo `/auth/` obrigatório**
- Billing: `GET /api/v1/stats/corporate/billing` (suporta `?period=YYYY-MM`)
- Cost Centers: `GET /api/v1/stats/corporate/cost-centers`
- Recent Rides: `GET /api/v1/stats/corporate/recent-rides`
- Cost Centers Usage: `GET /api/v1/stats/corporate/cost-centers-usage`
- Sempre espera `company_id` do usuário autenticado no backend
- Ver árvore completa de rotas em [ROUTES_TREE.md](../../podium-backend-api/ROUTES_TREE.md)

### Monorepo Context
- TypeScript config herda de nível superior (`../tsconfig.base.json`)
- Package `web-corporate` dentro de estrutura monorepo maior

## Patterns Específicos

### Error Handling em Hooks
```typescript
// Sempre mostrar Alert em UI + fallback mockado (ver useDashboard.ts)
catch (err: any) {
  setError(err.response?.data?.detail || "Erro ao conectar");
  setStats({ /* dados mockados */ });
}
```

### Formatação de Moeda
```typescript
// Sempre usar Intl.NumberFormat pt-BR (ver Dashboard.tsx)
const formatCurrency = (value: number) =>
  new Intl.NumberFormat("pt-BR", { style: "currency", currency: "BRL" }).format(value);
```

### Layout Consistency
- Páginas sempre usam `Box` container raiz
- Títulos: `Text textStyle="h1"` com cor white
- Grids: `SimpleGrid` com breakpoints `{ base: 1, md: 2, lg: 4 }`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WellingtonADS)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WellingtonADS)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
