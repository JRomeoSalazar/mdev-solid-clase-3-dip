# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A teaching exercise (class 3 of a SOLID course, in Spanish) on the Dependency Inversion Principle. `UserRegistrationService` is deliberately written with hidden dependencies, and the exercise is to refactor it. `README.md` holds the full assignment (Parts 1–4), acceptance criteria and hints; read it before changing anything.

## Commands

- `npm test` — run all tests (vitest); single file: `npx vitest run test/<file>.test.ts`; single test: `npx vitest run -t "<test name>"`
- `npm run typecheck` — `tsc --noEmit`
- `npm run demo` — runs `src/demo.ts` with `NODE_ENV=test`. Running it without that env var fails on purpose (infra refuses to work outside "test mode") until the exercise's Part 4 is done.

## Architecture

- `src/registration/user-registration-service.ts` — the use case (`register`, `confirm`). Currently instantiates `PostgresUserRepository`, `SmtpEmailSender`, `PasswordHasher` with `new`, and also uses the `auditLogger` singleton, `isFeatureEnabled` (env flags), `new Date()`/`Date.now()` and `Math.random()` directly.
- `src/registration/create-registration-service.ts` — intended composition root; the only place that should name concrete classes. Existing tests build the service only through `createRegistrationService()`.
- `src/infra/` — concrete implementations. The Postgres repository and SMTP sender keep module-level in-memory arrays and expose `__allRows`, `__sentEmails` and `__reset*` helpers; they throw unless `NODE_ENV === 'test'`.
- `src/types.ts` — shared data types (`RegistrationInput`, `StoredUser`, `RegistrationResult`).
- `test/user-registration-service.test.ts` — the original suite (12 tests) relying on fake timers, `Math.random` spying, `process.env` and `__reset*` calls. **Must not be modified**; it must keep passing after the refactor.

## Exercise constraints (from README acceptance criteria)

- Abstractions (e.g. `UserRepository`, `NotificationSender`, `Clock`, token/id generators) belong in `src/registration/` (or `src/domain/`), owned by the use case; `src/infra/` imports them, never the reverse.
- `src/registration/` must not import anything from `src/infra/`.
- No `process.env`, `new Date()` or `Math.random()` in `src/` outside infra implementations.
- New tests go in `test/user-registration-service.solid.test.ts` using hand-written test doubles (an `InMemoryUserRepository` living in tests): no `vi.useFakeTimers`, `vi.spyOn`, `process.env` or `__reset*`; each test builds its own service.
- No DI framework/container; constructor injection only.

## Conventions

- ESM TypeScript (Node >= 22); imports use explicit `.ts` extensions.
- Code comments and README are in Spanish; per the global rules, new docs should be written in English.
