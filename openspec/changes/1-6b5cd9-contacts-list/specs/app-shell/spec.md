# Spec Delta

## ADDED Requirements

### Requirement: The navigation links to the contacts screen
The UI's primary navigation SHALL include a "Contacts" link to `/contacts`, marked as the current page while the contacts screen is open.

#### Scenario: [app-shell-8bc928] The Contacts link opens the contacts screen
- **WHEN** a signed-in user on the home screen follows the "Contacts" link in the navigation
- **THEN** the page is at `/contacts` inside the shared layout, with the "Contacts" link marked as the current page and the "Contacts" heading in the main area
