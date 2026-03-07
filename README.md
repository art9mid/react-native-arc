# react-native-arc

Architecture reference and AI agent skill for scaffolding production-ready React Native (Expo) projects.

## What This Is

A collection of architecture guidelines, code templates, and a mobile design system that any AI coding agent can use to scaffold and maintain React Native projects with consistent, battle-tested patterns.

## Agent Commands

The skill is split into focused agents that can be run independently:

| Command | What It Does |
|---------|-------------|
| `/arc-scaffold` | Project init, folder structure, deps, config, linting, navigation |
| `/arc-connect` | API client, storage, state management, auth, i18n |
| `/arc-ui` | Theme system, components, screen content, mobile UX |
| `/arc-feature` | Add a complete new domain (types + API + hooks + screens) |
| `/arc-audit` | Check for UX, performance, and architecture violations |
| `/react-native-arc` | Full scaffolding (all-in-one) |

**Typical workflow:**
```
/arc-scaffold → /arc-connect → /arc-ui → /arc-feature (repeat) → /arc-audit
```

## How to Use

1. Clone or copy this repo into your project (or reference it)
2. Add the contents of `AGENTS.md` to your agent's instruction file:

| Agent | Instruction File |
|-------|-----------------|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` or `.cursor/rules` |
| Windsurf | `.windsurfrules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cline | `.clinerules` |
| Aider | `.aider.conf.yml` |
| Other | Whatever your agent reads — paste from `AGENTS.md` |

The key line to add is: **"Read the skill files in `skills/react-native-arc/` before generating code."**

## Architecture Stack

| Layer | Solution |
|-------|----------|
| Framework | React Native + Expo (TypeScript strict) |
| Navigation | React Navigation 7.x (native-stack + bottom-tabs) |
| Server State | TanStack React Query + MMKV persistence |
| HTTP | Axios with auth interceptors + automatic token refresh |
| Storage | react-native-mmkv |
| Styling | Custom theme system + `useStyles()` + `react-native-size-matters` |
| Images | expo-image (blurhash + disk cache) |
| Animations | react-native-reanimated (UI thread) |
| Forms | Formik + Yup |
| i18n | Lingui.js + expo-localization (optional) |

## Project Structure

```
skills/react-native-arc/
├── project-structure.md          # Folder tree, naming, barrel exports
├── navigation.md                 # Navigation setup, typed hooks, deep linking
├── theme.md                      # Theme system, color schemes, OLED dark mode
├── components.md                 # Component architecture, touch targets, haptics
├── api-services.md               # HTTP client, API modules, error handling
├── storage.md                    # MMKV storage wrappers
├── state-management.md           # React Query setup, query/mutation patterns
├── performance.md                # FlatList, images, animations, startup
├── providers.md                  # Provider stack, AppInitialization
├── typescript.md                 # TSConfig, path aliases, conventions
├── linting.md                    # ESLint, Prettier, Husky, lint-staged
├── i18n.md                       # Internationalization with Lingui
├── mobile-design/                # Mobile UX design system
│   ├── GUIDE.md                  # Master checklist & anti-patterns
│   ├── touch-psychology.md       # Fitts' Law, thumb zones, haptics
│   ├── mobile-performance.md     # Deep optimization guide
│   ├── mobile-navigation.md      # Tab bar, stack, deep linking UX
│   ├── mobile-typography.md      # System fonts, Dynamic Type, scales
│   ├── mobile-color-system.md    # OLED, dark mode, contrast
│   ├── platform-ios.md           # iOS HIG specifics
│   ├── platform-android.md       # Material Design 3 specifics
│   ├── mobile-backend.md         # Offline sync, push, auth patterns
│   ├── decision-trees.md         # Framework & architecture decisions
│   ├── mobile-testing.md         # Testing pyramid & tools
│   ├── mobile-debugging.md       # Debugging tools & techniques
│   └── scripts/mobile_audit.py   # 50+ automated mobile UX checks
└── templates/
    ├── component.md              # Component scaffolding template
    ├── screen.md                 # Screen scaffolding template
    ├── hook.md                   # Custom hook template
    └── api-service.md            # API service + React Query template
```

## Component Convention

```
component-name/
├── index.tsx                    # Component + barrel export
├── component-name.styles.ts     # Styles via useStyles callback
├── component-name.types.ts      # Props interface
├── component-name.constants.ts  # (optional) Variants, sizes
├── component-name.utils.ts      # (optional) Pure helpers
├── component-name.hooks.ts      # (optional) Component hooks
└── components/                  # (optional) Sub-components
    ├── index.ts
    └── sub-component/
        └── index.tsx
```

## Mobile Audit Script

Run automated UX checks on any React Native project:

```bash
python skills/react-native-arc/mobile-design/scripts/mobile_audit.py /path/to/project
```

## Context7 (Optional)

If using Claude Code, you can add Context7 MCP for latest library docs during scaffolding:

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp@latest
```

## License

MIT
