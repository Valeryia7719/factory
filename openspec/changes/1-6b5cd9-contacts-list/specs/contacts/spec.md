# Spec Delta

## Purpose

The people in the CRM: the `contacts` query that returns them and the screen that lists them. Every signed-in user sees the same contacts.

## ADDED Requirements

### Requirement: The contacts query lists every contact newest first
The API's `contacts` query SHALL answer a signed-in request with every contact, each with its id, name, email and created date, the most recently created first, and an empty list when there are none.

#### Scenario: [contacts-ced097] A signed-in request reads the contacts newest first
- **WHEN** a signed-in client sends `{ contacts { id name email createdAt } }` and several contacts exist
- **THEN** it gets every contact, ordered from the most recently created to the oldest
- **NO E2E** not-built: a contact cannot be created until the "add a contact" issue; proven by services/backend/tests/integration/contacts.test.ts

#### Scenario: [contacts-19dbc8] A signed-in request with no contacts gets an empty list
- **WHEN** a signed-in client sends `{ contacts { id } }` and no contact exists
- **THEN** it gets `contacts` as an empty list and no error

### Requirement: Only a signed-in user reads the contacts
The API SHALL refuse the `contacts` query to a request without a valid session with a GraphQL error whose `extensions.code` is `UNAUTHENTICATED`, and no contacts.

#### Scenario: [contacts-ad0e29] A signed-out request is refused
- **WHEN** a client without a valid session sends the `contacts` query
- **THEN** the answer has `data: null` and an error with `extensions.code` `UNAUTHENTICATED`

### Requirement: The contacts screen lists each contact
The UI SHALL show a signed-in user, at `/contacts`, a "Contacts" screen that lists each contact's name and email in the order the `contacts` query answers, with the navigation link "Contacts" marked current.

#### Scenario: [contacts-fe490f] The contacts screen lists the API's contacts
- **WHEN** a signed-in user opens `/contacts` and the `contacts` query answers several contacts
- **THEN** the page shows the "Contacts" heading and each contact's name and email in the answered order
- **NO E2E** not-built: a contact cannot be created until the "add a contact" issue; proven by services/ui/tests/integration/contacts-page.test.tsx

### Requirement: The contacts screen says when there are no contacts
The UI SHALL show "No contacts yet." on the contacts screen when the `contacts` query answers an empty list.

#### Scenario: [contacts-884e1e] No contacts shows the empty state
- **WHEN** a signed-in user opens `/contacts` and no contact exists
- **THEN** the page shows the "Contacts" heading and "No contacts yet.", and no list of contacts

### Requirement: A contacts screen that fails to load says so
The UI SHALL replace the list, when the `contacts` query fails or answers something else, with a "Couldn't load contacts." notice that offers to reload.

#### Scenario: [contacts-45349f] A failed load shows the notice
- **WHEN** the `contacts` request fails or its answer is malformed
- **THEN** the page shows "Couldn't load contacts." and a Reload button, and no list of contacts
