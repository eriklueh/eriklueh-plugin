# Zustand en Next.js 16 App Router

## Dos casos, dos patrones
1. **Estado global de UI puro** (tema, sidebar abierto/cerrado): un singleton a nivel de modulo esta bien.
   ```ts
   export const useUIStore = create<UIState>()((set) => ({ ... }))
   ```
2. **Estado inicializado con datos por request** (p.ej. sembrado desde un Server Component): NO crear el store a nivel de modulo — en SSR/RSC un singleton de modulo se comparte entre requests y **filtra estado entre usuarios**. Usar store-per-request con Context Provider.

## Patron store-per-request + Provider
```ts
// stores/cart-store.ts
import { createStore } from 'zustand/vanilla'
export type CartState = { items: Item[]; add: (i: Item) => void }
export const createCartStore = (init: Partial<CartState> = {}) =>
  createStore<CartState>()((set) => ({
    items: [], add: (i) => set((s) => ({ items: [...s.items, i] })), ...init,
  }))
```
```tsx
// stores/cart-provider.tsx
'use client'
import { createContext, useContext, useRef } from 'react'
import { useStore } from 'zustand'
import { createCartStore, type CartState } from './cart-store'

const CartCtx = createContext<ReturnType<typeof createCartStore> | null>(null)

export function CartProvider({ children, initial }: { children: React.ReactNode; initial?: Partial<CartState> }) {
  const ref = useRef<ReturnType<typeof createCartStore>>()
  if (!ref.current) ref.current = createCartStore(initial)   // una instancia por montaje/request
  return <CartCtx.Provider value={ref.current}>{children}</CartCtx.Provider>
}

export function useCart<T>(selector: (s: CartState) => T): T {
  const store = useContext(CartCtx)
  if (!store) throw new Error('useCart debe usarse dentro de <CartProvider>')
  return useStore(store, selector)
}
```
El Server Component pasa los datos iniciales por props a `<CartProvider initial={...}>`.

## Hidratacion con `persist`
`persist` solo aplica a estado de cliente (localStorage); nunca persistir datos del servidor. Para evitar mismatches de hidratacion en App Router:
- Usar un flag `_hasHydrated` + `onRehydrateStorage`, y no renderizar el estado persistido hasta que hidrate; o
- `skipHydration: true` y llamar a `store.persist.rehydrate()` en un `useEffect`.
- `partialize` para persistir SOLO los campos elegidos (excluir `isLoading`, `error`, acciones).
