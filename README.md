# CognitiveStack

![CognitiveStack](CognitiveStack.png)

Software engineering specifications, prompt architectures, and modular skills for building
agentic systems.

`agent-skills` · `prompt-engineering` · `system-architecture` · `llm` · `workflows`

## Que es

CognitiveStack es un sistema de conocimiento en Markdown que un proyecto instala junto a su
codigo para que los agentes de IA sepan como trabajar antes de modificar nada.

No es un starter de codigo ni una libreria. Es la capa de contexto: especificaciones de
ingenieria, limites arquitectonicos, criterios de interfaz, reglas de calidad y seguridad, y
skills reutilizables, escritos para que Codex, Claude, GitHub Copilot u otro agente lean lo
mismo y lleguen a las mismas decisiones.

## Por que existe

Un agente genera codigo rapidamente, pero sin contexto persistente suele:

- inventar patrones distintos en cada modulo;
- mezclar reglas de negocio con infraestructura;
- duplicar componentes y documentacion;
- construir interfaces desde el esquema de base de datos;
- exponer detalles tecnicos en pantalla;
- omitir estados, permisos, pruebas o migraciones;
- depender de conversaciones que el siguiente agente no conoce.

CognitiveStack convierte esas decisiones en instrucciones versionadas, navegables y
verificables dentro del propio repositorio.

## Que contiene

| Area           | Contexto incluido                                                     |
| -------------- | --------------------------------------------------------------------- |
| Agentes        | Instrucciones para Codex, Claude y GitHub Copilot                     |
| Arquitectura   | Express.js, Next.js, limites por capas e integraciones                |
| Datos          | PostgreSQL, Prisma y migraciones versionadas                          |
| Autenticacion  | Better Auth y reglas para no reinventar sesiones                      |
| Frontend       | shadcn/ui, Tailwind, formularios, tablas y estados compartidos        |
| Criterio UX    | Brief de pantalla, niveles de informacion y presupuesto de acciones   |
| Diseno visual  | Tokens, escalas de espaciado, tipografia, color y motion              |
| Calidad        | Definition of Done, testing, code review y observabilidad             |
| Rendimiento    | Presupuestos, medicion y anti-patrones                                |
| Seguridad      | Principios, hardening por frontera y threat model                     |
| Ejecucion      | Planes activos, decisiones y skills reutilizables                     |
| Entrega        | Commits pequenos, mensajes descriptivos y pull requests               |

### Skills

Procedimientos que se cargan solo cuando la tarea los necesita, en `.agents/skills/`:

`implement-feature` · `implement-operational-frontend` · `create-migration` · `review-code`
· `harden-security` · `optimize-performance` · `update-documentation` · `commit-changes`

## Stack de referencia

Node.js y TypeScript, Express.js para la API REST, Next.js y React en el frontend, Better
Auth para identidad y sesiones, PostgreSQL con Prisma, Tailwind CSS y shadcn/ui, React Hook
Form con Zod, Axios para integraciones server-side, pnpm mediante Corepack y Docker para
desplegar.

El stack es una decision base, no una restriccion irreversible. Un proyecto que necesita
desviarse registra la razon y sus consecuencias en un ADR.

## Como funciona el contexto

```text
AGENTS.md
  -> indica que leer y como trabajar
ARCHITECTURE.md
  -> muestra componentes, limites y dependencias
docs/product/
  -> explica que debe hacer el producto
docs/architecture/
  -> explica como debe construirse
docs/plans/
  -> conserva la estrategia de trabajos complejos
.agents/skills/
  -> ejecuta procedimientos repetibles
docs/quality/ + docs/security/
  -> define cuando un cambio es aceptable
```

La idea no es cargar todo el repositorio en cada prompt. El agente empieza con un mapa
pequeno y abre solamente la fuente de verdad relevante para la tarea.

## Criterio de interfaz

La parte mas desarrollada del contexto evita que un panel operativo termine como una
acumulacion de cards, filtros y mensajes. Disenar es decidir que no aparece:

- responder por escrito quien usa la pantalla, que decide, con que informacion y cual es su
  unica accion primaria, antes de implementarla;
- disenar desde el trabajo del usuario, nunca desde el esquema de base de datos;
- no mostrar un dato solo porque el backend lo devuelve;
- clasificar la informacion en P0, P1, P2 y P3, y dejar lo tecnico fuera de la vista inicial;
- limitar las acciones simultaneas: una primaria, una o dos secundarias, el resto revelado;
- tomar espaciado, tipografia, color, radios y motion de escalas documentadas;
- cubrir todos los estados: carga, vacio, parcial, error, permiso y exito;
- verificar teclado, movil, light y dark antes de dar algo por terminado.

## Mapa del repositorio

```text
.
├── AGENTS.md                       # Reglas permanentes para agentes
├── ARCHITECTURE.md                 # Mapa tecnico de alto nivel
├── CLAUDE.md                       # Entrada para Claude
├── .agents/skills/                 # Procedimientos reutilizables
├── .github/                        # Instrucciones para Copilot
├── .codex/                         # Configuracion, no conocimiento del producto
└── docs/
    ├── product/                    # Producto, dominio y features
    ├── architecture/               # Stack, boundaries y patrones
    ├── plans/                      # Trabajo activo, completado y deuda
    ├── quality/                    # Testing, review, commits, rendimiento y DoD
    ├── security/                   # Principios, hardening y amenazas
    └── generated/                  # Referencias derivadas
```
