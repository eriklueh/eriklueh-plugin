---
name: zustand-patterns
description: Patrones de estado con Zustand en esta app Next.js 16. Usar al crear o modificar un store, compartir estado de cliente entre componentes, o decidir entre estado de servidor y de cliente.
---

# Zustand — patrones del proyecto

## Cuando usar Zustand (y cuando NO)
- Usalo para estado de UI de cliente: modales, filtros, pasos de un wizard, preferencias, carrito.
- NO lo uses para datos del servidor: eso vive en la base de datos (Drizzle) y se pasa por props desde Server Components o se recarga. Zustand no es una cache de datos remotos (para eso, revalidacion de Next o una libreria de fetching).

## Estructura
- Un store por dominio en `stores/<dominio>.ts`. Evitar un unico mega-store global.
- Hook nombrado `useXStore`, con las acciones dentro del store:
  ```ts
  import { create } from 'zustand'

  interface CartState {
    items: Item[]
    add: (i: Item) => void
    clear: () => void
  }

  export const useCartStore = create<CartState>((set) => ({
    items: [],
    add: (i) => set((s) => ({ items: [...s.items, i] })),
    clear: () => set({ items: [] }),
  }))
  ```

## Reglas
- Selectores atomicos para evitar re-renders: `const items = useCartStore((s) => s.items)`. No desestructurar el store entero.
- Las mutaciones van en acciones del store, no con setState suelto en los componentes.
- `'use client'` obligatorio en cualquier componente que consuma el store.
- Persistencia solo si hace falta: middleware `persist` con `name` explicito. Cuidado con la hidratacion en Next (usar un guard de `hasHydrated` o `skipHydration`).
- Nunca poner logica de servidor ni secretos en un store (todo el store viaja al cliente).

## SSR / Next 16
- Para estado global de UI, el singleton a nivel de modulo esta bien.
- Si necesitas estado inicializado por request (con datos del servidor), no crees el store a nivel de modulo: usa un provider con `createStore` + Context para tener una instancia por request y evitar filtrar estado entre usuarios.
