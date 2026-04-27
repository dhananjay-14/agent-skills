---
name: unit-testing
description: >-
  Write unit tests for services, repositories, controllers, and components.
  Use when asked to write tests, create spec files, add test coverage, or test
  a feature. Covers module setup, mocking patterns, test data, and conventions.
---

# Unit Testing Rules

## Environment & Execution

- Confirm the correct runtime version before running tests (check any version manager config file in the repo).
- Use the project's designated package manager — do not substitute an alternative.
- Typical test commands to look for in `package.json`:
  - Run all tests
  - Run a single spec file by path pattern
  - Watch mode
  - Coverage mode

## File Placement & Naming

- Place spec files **colocated** next to the file under test — same directory, same base name.
- Use the `.spec.ts` extension. Never `.test.ts` or `.e2e-spec.ts`.

## Test Data & Fixtures

- Check the project for existing fixture files or shared mock data before creating new ones — reuse what exists.
- Use realistic IDs and enum values from the project's own enums — never hardcode magic strings or status values.
- Scope fixtures appropriately: shared/generic fixtures belong in a common mock location; domain-specific fixtures belong near the domain they test.

## Mocking Patterns by Layer

### Controllers
- Auto-mock the service the controller depends on.
- Verify the controller delegates correctly to the service — assert both the call arguments and the return value.

### Services (Feature / Business Logic layer)
- Mock **all** injected dependencies — never let real implementations run in unit tests.
- Use `jest.spyOn` to control return values per test.
- Clear and reset all mocks in `afterEach` to prevent state leaking between tests.

### Repositories (Data layer)
- Provide a mock model/repository class instead of a real database connection.
- Spy on repository methods and assert they are called with the correct query parameters.
- Test both the found and not-found paths.

### External / HTTP Services
- Provide manual `useValue` mocks for HTTP clients, config services, and utility wrappers.
- Match the interface of the real dependency exactly so type checking still applies.

### Components with Many Dependencies
- Same approach as services — mock every injected dependency individually.
- Use domain-specific fixture files colocated with the component.

## Test Structure Conventions

- Always include a `should be defined` smoke test as the first case.
- Group tests by method name using nested `describe` blocks.
- Label cases clearly: happy path, error/sad path, edge/boundary case.
- Use `mockResolvedValueOnce` (not `mockResolvedValue`) to prevent mock return values from leaking across tests.
- Always `clearAllMocks` and `resetAllMocks` in `afterEach`.

## Assertions

- Use the test framework's built-in matchers — avoid third-party assertion libraries unless the project already uses one.
- For each test, assert both **what was called** (spy assertions) and **what was returned** (value assertions).
- For async error paths, assert that the promise rejects with the expected error type.

## Common Pitfalls

- **Decorator side effects** — framework decorators (caching, locking, error suppression) may wrap methods. If a decorator interferes with a test, mock its underlying dependency or test the inner logic directly.
- **Fire-and-forget calls** — background async calls won't surface errors in tests, but you can still assert they were invoked with the correct arguments.
- **Global test setup** — check for global setup/teardown files that handle infrastructure (e.g. in-memory databases). Do not re-mock things already handled globally.
- **Import paths** — follow the project's existing convention for import paths (absolute vs relative) to avoid resolution errors.

## Checklist Before Submitting

- [ ] Spec file is colocated next to the source file
- [ ] Named `*.spec.ts`
- [ ] Every public method has at least a happy path and an error path test
- [ ] All injected dependencies are mocked
- [ ] `afterEach` clears and resets mocks
- [ ] Test runs successfully with no warnings or unhandled rejections