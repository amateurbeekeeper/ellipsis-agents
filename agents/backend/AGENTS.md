# Backend Agent — Rules

## Repository
https://github.com/amateurbeekeeper/ellipsisandco

## Your working directory
`apps/api/src/app/`

## Module pattern (always follow this)
Every domain: `<domain>.module.ts`, `<domain>.service.ts`, `<domain>.controller.ts`, `entities/<name>.entity.ts`, `dto/<action>-<name>.dto.ts`, `<domain>.service.spec.ts`

## Test pattern
```typescript
const mockRepo = () => ({
  findOne: jest.fn(), save: jest.fn(), create: jest.fn((x) => x),
  update: jest.fn(), delete: jest.fn(), find: jest.fn(),
});
```
- Unit tests mock all repos — no DB connection needed
- Cover: happy path, not-found, conflict, expired, unauthorized

## Commands (always use nx)
```bash
nx test api-main          # run tests
nx lint api-main          # lint
nx typecheck api-main     # typecheck
nx build api-main         # build
```

## PR format
Title: concise lowercase, max 72 chars, no period
Body: bullet points of what was done + link to relevant GitHub issue
Branch: `agent/backend/<short-description>`

## GitHub issues
Track work at: https://github.com/amateurbeekeeper/ellipsisandco/issues
Comment on issues as you work. Close when done.
