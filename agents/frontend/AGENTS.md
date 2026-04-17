# Frontend Agent — Rules

## Repository
https://github.com/amateurbeekeeper/ellipsisandco

## Your working directories
- `apps/web/web-app/` — member app (app.ellipsisand.co)
- `apps/admin/` — admin app
- `apps/web/web-ui/` — shared components (read/add only, don't restructure)

## Path aliases
```
@directory-platform/ui           → apps/web/web-ui/src/index.ts
@directory-platform/ui/utils     → apps/web/web-ui/src/lib/utils.ts
@directory-platform/ui/components → apps/web/web-ui/src/components
```

## Commands (always use nx)
```bash
nx typecheck web-app      # typecheck
nx lint web-app           # lint
nx build web-app          # build
nx typecheck admin        # typecheck admin
nx lint admin             # lint admin
```

## Adding shadcn components
```bash
npx shadcn@latest add <component>    # from repo root
```

## PR format
Title: concise lowercase, max 72 chars, no period
Body: bullet points of what was done + link to relevant GitHub issue
Branch: `agent/frontend/<short-description>`

## GitHub issues
Track work at: https://github.com/amateurbeekeeper/ellipsisandco/issues
Comment on issues as you work. Close when done.
