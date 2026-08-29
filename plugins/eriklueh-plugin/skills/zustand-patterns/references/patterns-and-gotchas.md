# Patrones y gotchas de Zustand (v5)

## Tipado con middleware: doble parentesis
Con TypeScript + middleware es obligatorio el curry `create<T>()(...)`:
```ts
// CORRECTO
export const useStore = create<State>()(devtools(persist(immer((set) => ({ ... })), { name: 'store' })))
// MAL: create<State>(...)  -> falla la inferencia de tipos con middleware
```
Orden de middleware, de fuera hacia dentro: `devtools(persist(immer(...)))`. Con ese orden las acciones de `persist` aparecen en Redux DevTools. Nombrar acciones: `set(next, undefined, 'nombreAccion')`.

## Acciones agrupadas
Agrupar todas las acciones bajo `actions` y exponer un hook:
```ts
interface State { count: number; actions: { inc: () => void; reset: () => void } }
export const useCounterActions = () => useStore((s) => s.actions)
```

## Selectores (evitar re-renders)
```ts
const count = useStore((s) => s.count)                 // atomico: OK
const { a, b } = useStore(useShallow((s) => ({ a: s.a, b: s.b }))) // multi-valor: useShallow
// MAL: useStore((s) => ({ a: s.a }))  -> objeto nuevo por render -> re-render infinito en v5
```
`useShallow` se importa de `zustand/react/shallow` (en v5 el 2º argumento de equality del hook ya no existe).

## Estado derivado
Calcular en el selector (`useStore((s) => s.items.length)`) en vez de guardar `total`. Para calculos caros: `useMemo` FUERA del store.

## Acciones async (gotchas criticos)
```ts
load: async (id) => {
  set({ loading: true, error: null })
  try {
    const data = await api.get(id)
    if (get().loading) set({ data, loading: false })  // guard: no pisar un estado ya cambiado por otra accion
  } catch (e) {
    set({ error: (e as Error).message, loading: false })
  }
}
```
- `set()` es sincrono, pero el re-render de React es batched: para leer el valor nuevo al instante usa `get()`/`store.getState()`, no esperes que el componente lo vea en el mismo tick.
- Tras cualquier `await`, relee con `get()`; nunca confies en valores capturados por closure (stale).

## Testing
```ts
const initialState = { count: 0 }
// dentro del store: reset: () => set(initialState)
beforeEach(() => useStore.getState().reset())
```

## Persistencia
`persist` con `name` unico (evita colisiones en localStorage), `version` + `migrate` para cambios de forma, y `partialize` para elegir que se guarda.
