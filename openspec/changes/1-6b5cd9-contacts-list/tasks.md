# Tasks

<!-- Plan written by superpowers:writing-plans. See design.md for the approach and the spec deltas in specs/ for every scenario ID. -->

Conventions for every step: run commands from the repository root; a service's tests run through its checks container, so "run the tests" means `bin/check <service> --gates coverage` (both suites), and the full gate run is `bin/check <service>`. Tests are written from the scenarios before the code they test. Commit subjects are `[1][6b5cd9] <scope>: <Description>`.

Review focus (each has a test below):
- A page that is still loading never shows "No contacts yet." (task 3, step 3.1).
- Two contacts created at the same instant keep a stable order (task 1, step 1.1).
- A contact name or email with markup renders as text (task 3, step 3.1).
- A malformed answer shows the failure notice, not an empty list (task 3, step 3.1).

## 1. Contacts table, migration and store

- [ ] 1.1 [backend] Write `services/backend/tests/integration/contacts-store.test.ts` over `migratedDatabase()` from `tests/support/database.ts`: "contacts are listed newest first" inserts three contacts with distinct `createdAt` and expects `list()` to return them newest first; "contacts created at the same instant are listed by id, descending" inserts two rows with equal `createdAt` and expects the larger id first; "an empty table lists nothing" expects `[]`. Each returned item is `{ id: string, name: string, email: string, createdAt: Date }`.
- [ ] 1.2 [backend] Run `bin/check backend --gates coverage`; expect it to fail because `src/db/contacts-store.ts` does not exist.
- [ ] 1.3 [backend] Add `src/db/schema/contacts.ts` and re-export it from `src/db/schema/index.ts`. Table `contacts`: `id uuid` primary key `defaultRandom()`, `name text` not null, `email text` not null, then `...timestamps` from `./timestamps`.
- [ ] 1.4 [backend] Emit the migration with the checks image so the host needs no Bun: build `docker build --target checks -t factory-backend-checks services/backend`, then run `docker run --rm -v "$PWD/services/backend/src:/app/src" factory-backend-checks bun run db:generate`, and finish with `bun run db:verify` the same way. Expect one new `src/db/migrations/<timestamp>_<name>/` folder holding a `CREATE TABLE "contacts"` statement and nothing that drops or alters existing tables.
- [ ] 1.5 [backend] Add `src/contacts.ts` exporting `type Contact = { id: string; name: string; email: string; createdAt: Date }` and `type ContactStore = { list: () => Promise<Contact[]> }`, with a header comment in the style of `src/dashboard.ts`. Add `src/db/contacts-store.ts` exporting `databaseContactStore(db: AnyDatabase): ContactStore`, ordering by `createdAt` descending then `id` descending.
- [ ] 1.6 [backend] Run `bin/check backend --gates coverage`; expect the new store tests to pass.
- [ ] 1.7 [backend] Commit: `git add services/backend && git commit -m "[1][6b5cd9] backend: Add the contacts table and store"`.

## 2. The contacts query

- [ ] 2.1 [backend] Extend `tests/support/server.ts` so a test can insert rows before the app starts: `startSeeded` takes an optional `seed?: (db: TestDatabase) => Promise<void>` that runs after the demo user exists, and the `createApp` call there passes `contacts: databaseContactStore(db)`.
- [ ] 2.2 [backend] Write `services/backend/tests/integration/contacts.test.ts` with the query `{ contacts { id name email createdAt } }`, parsed with zod like `dashboard.test.ts`: `[contacts-ced097]` seeds three contacts through `seed` and expects them newest first, each `createdAt` an ISO-8601 string; `[contacts-19dbc8]` a signed-in request on a fresh app expects `contacts: []` and no `errors`; `[contacts-ad0e29]` `test.each([undefined, "better-auth.session_token=forged"])` expects the same `UNAUTHENTICATED` answer shape `dashboard.test.ts` uses.
- [ ] 2.3 [backend] Run `bin/check backend --gates coverage`; expect failures because the field does not exist.
- [ ] 2.4 [backend] Wire it: add `contacts: ContactStore` to `Context` in `src/graphql/builder.ts`, to `createApp`'s `deps` and its context object in `src/app.ts`, and pass `contacts: databaseContactStore(db)` in `src/server.ts`. Add `src/graphql/resources/contacts.ts` defining `Contact` (`id` as `ID`, `name`, `email`, `createdAt` as an ISO-8601 string) and the root field `contacts: [Contact!]!` that calls `viewerRequired(context)` and returns `context.contacts.list()`; import it in `src/graphql/schema.ts` beside the other resources.
- [ ] 2.5 [backend] Run `bin/contracts generate`; expect `services/backend/contracts/schema.graphql` to gain `type Contact` and `contacts: [Contact!]!`, and the UI's generated folder to stay unchanged until task 3 adds its query.
- [ ] 2.6 [backend] Run `bin/check backend`; expect every gate to pass, including coverage at 100% of lines and functions.
- [ ] 2.7 [backend] Commit: `git add services/backend && git commit -m "[1][6b5cd9] backend: Add the contacts query"`.

## 3. The contacts screen, route and navigation

- [ ] 3.1 [ui] Write the tests first. `tests/support/contacts.ts` exports `contactsAnswer(contacts: unknown[], delay = 20): MockLink.MockedResponse` and `contactsOffline`, shaped like `tests/support/dashboard.ts`, over the `ContactsQuery` document of step 3.3. `tests/unit/hooks/useContacts.test.tsx` covers loading, ready, a network error, a malformed answer, and an unconfigured API, as `useDashboard.test.tsx` does. `tests/unit/pages/contacts/ContactsPage.test.tsx` covers `[contacts-fe490f]` (the heading and each name and email in the answered order), `[contacts-884e1e]` ("No contacts yet." and no list), `[contacts-45349f]` ("Couldn't load contacts." and a Reload button that calls `window.location.reload`), no "No contacts yet." while the query is loading, a name such as `<b>Ann</b>` shown as literal text, and a malformed answer showing the failure notice. `tests/unit/components/AppHeader.test.tsx` gains `[app-shell-8bc928]`: a "Contacts" link to `/contacts`, marked `aria-current="page"` on `/contacts` and not on `/`. `tests/integration/routes.test.tsx` gains a case that `/contacts` renders inside the layout.
- [ ] 3.2 [ui] Run `bin/check ui --gates coverage`; expect failures for the missing hook, page, route and link.
- [ ] 3.3 [ui] Add `src/services/graphql/contacts.ts` exporting `ContactsQuery = graphql(\`query Contacts { contacts { id name email } }\`)`, then run `bin/contracts generate` to refresh `src/services/graphql/generated/`. Add `src/hooks/useContacts.ts` exporting `useContacts(): ContactsState` where `ContactsState` is `{ status: "loading" } | { status: "failed" } | { status: "ready"; contacts: Contact[] }`, validating the answer with zod as `useDashboard` does.
- [ ] 3.4 [ui] Add `src/pages/contacts/ContactsPage.tsx` exporting `ContactsPage(): ReactElement`: an `h1` "Contacts"; a list of each contact's name and email; "No contacts yet." only once ready with an empty list; the `Notice` with a Reload button on failure, as `DashboardPage` does. Add `contacts.title`, `contacts.empty`, `contacts.failed` ("Couldn't load contacts.") and `contacts.retry` ("Reload to try again.") plus `nav.contacts` ("Contacts") to `src/i18n/locales/en/common.json`.
- [ ] 3.5 [ui] Add the `/contacts` route beside the index route in `src/app/routes.tsx`, and a `NavLink` to `/contacts` after the Dashboard one in `src/components/AppHeader.tsx`, with the same classes.
- [ ] 3.6 [ui] Run `bin/check ui`; expect every gate to pass, including the i18n-keys test and coverage at 100%.
- [ ] 3.7 [ui] Commit: `git add services/ui && git commit -m "[1][6b5cd9] ui: Add the contacts screen and navigation link"`.

## 4. End-to-end tests

- [ ] 4.1 [e2e] Write `services/e2e/tests/contacts.spec.ts`, titles starting with the scenario ID. API tests with `demoSession()` and `graphql()` from `tests/fixtures/api.ts`: `[contacts-19dbc8]` a signed-in client reads `contacts` as an empty list; `[contacts-ad0e29]` a signed-out client gets `data: null` and `UNAUTHENTICATED`. Browser tests with `signIn` from `tests/fixtures/auth.ts`: `[contacts-884e1e]` open `/contacts`, see the "Contacts" heading and "No contacts yet." in the main area; `[contacts-45349f]` abort the `Contacts` operation with `page.route`, reload, see "Couldn't load contacts.", "Reload to try again." and the Reload button. Add `[app-shell-8bc928]` to `tests/app-shell.spec.ts`: from `/` click the "Contacts" link in the navigation, expect the URL to end in `/contacts`, the link `aria-current="page"` and the heading. A comment on the empty-state tests says the suite runs on a fresh stack and the "add a contact" issue must update them.
- [ ] 4.2 [e2e] Run `bin/e2e --gates`; expect lint and typecheck to pass.

## 5. Prove it on the running stack

- [ ] 5.1 [repo] Run `bin/up` to rebuild and restart this worktree's stack, then `bin/e2e --changed`; expect every test of the changed capabilities to pass, including the six new scenario IDs.
- [ ] 5.2 [repo] Drive the Orca browser through the `orca-cli` skill: sign in as `demo@example.com`, click "Contacts", confirm the empty state, and send a signed-out `contacts` query with curl to confirm `UNAUTHENTICATED`.
- [ ] 5.3 [repo] Capture the proof: `bin/e2e --proof --change 1-6b5cd9-contacts-list -- --grep "contacts-19dbc8|contacts-ad0e29|contacts-884e1e|contacts-45349f|app-shell-8bc928"`, commit the screenshots under `openspec/changes/1-6b5cd9-contacts-list/proof/` with `[1][6b5cd9] e2e: Add contacts proof`.
- [ ] 5.4 [repo] Run `bin/check backend ui`, `bin/e2e` in full and `bin/openspec validate 1-6b5cd9-contacts-list --strict`; expect every one to pass. `/work-pr` then verifies, archives and ships.
