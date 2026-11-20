# Proposal

**Issue:** #1
**Type:** feature
**Services:** backend, ui, e2e

## Why

The CRM has no people in it yet, and companies, deals and notes all hang off contacts. The app first needs somewhere to keep contacts and a screen that shows them.

## What Changes

- A `contacts` table (id, name, email, created and updated dates) with an expand-only migration.
- A `contacts` GraphQL query that returns every contact, newest first, to a signed-in user and refuses a signed-out request with `UNAUTHENTICATED`; the contract is regenerated.
- A "Contacts" link in the primary navigation and a `/contacts` page that lists each contact's name and email, says "No contacts yet." when there are none, and shows a "Couldn't load contacts." notice with a Reload button when the query fails.
- End-to-end tests for the empty state, the API's empty list and the signed-out refusal; the populated list is covered by backend and UI integration tests.

Adding, editing, deleting, sorting, searching and paging contacts are out of scope.

## Capabilities

### New Capabilities
- `contacts`: the people in the CRM; this change adds the `contacts` query, its signed-in-only rule and the contacts list page with its empty and failure states.

### Modified Capabilities
- `app-shell`: the primary navigation gains a "Contacts" link to the new `/contacts` route.

## Impact

- `services/backend`: a new table and migration, a contact store, a GraphQL resource and the regenerated `contracts/schema.graphql`.
- `services/ui`: a contacts page, hook, query document, route, navigation link and English strings.
- `services/e2e`: a `contacts` spec file citing the new scenarios.
- No new dependencies.
