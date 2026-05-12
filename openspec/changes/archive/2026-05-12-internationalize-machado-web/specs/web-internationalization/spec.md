## ADDED Requirements

### Requirement: Web locale configuration is centralized
The `machado-web` application SHALL provide a centralized internationalization configuration that defines supported locales, fallback behavior, translation resources, and locale resolution rules without requiring changes to `machado-api`.

#### Scenario: Supported locales are defined in one place
- **WHEN** a maintainer adds or updates a supported UI language
- **THEN** the locale identifier, label, and translation resource mapping MUST be maintained in a single frontend locale configuration module

#### Scenario: Backend remains unchanged
- **WHEN** internationalization is implemented in `machado-web`
- **THEN** the change MUST NOT require new API endpoints, backend contract changes, or locale-specific behavior from `machado-api`

### Requirement: Initial locale resolution follows deterministic priority
The `machado-web` application SHALL resolve the active locale using deterministic priority to keep initial language behavior predictable across sessions.

#### Scenario: Persisted locale has highest priority
- **WHEN** the user has a previously saved locale and that locale is supported
- **THEN** the application MUST initialize the interface using the saved locale

#### Scenario: Browser locale is used when no saved locale exists
- **WHEN** the user has no saved locale and the browser default language maps to a supported locale
- **THEN** the application MUST initialize the interface using the supported browser locale

#### Scenario: Unsupported browser locale falls back to en
- **WHEN** the user has no saved locale and the browser default language is not supported
- **THEN** the application MUST initialize the interface using `en`

#### Scenario: Invalid saved locale falls back safely
- **WHEN** the persisted locale value is missing, malformed, or unsupported
- **THEN** the application MUST ignore that value and continue locale resolution using browser locale and then fallback `en`

### Requirement: Users can change locale from the navbar
The `machado-web` application SHALL allow users to manually change the interface language from the navbar using a dedicated language selector.

#### Scenario: Navbar exposes supported language options
- **WHEN** the user opens or focuses the language selector in the navbar
- **THEN** the selector MUST offer `pt-BR`, `en`, and `es` options identified with Brazil, United States, and Spain flag visuals plus a readable language label or abbreviation

#### Scenario: Manual language switch updates visible text
- **WHEN** the user selects a different supported locale from the navbar
- **THEN** the application MUST update visible fixed interface text to the selected locale without requiring backend changes

#### Scenario: Active locale is visually indicated
- **WHEN** the current locale matches one of the selector options
- **THEN** the selector MUST clearly indicate which locale is active

### Requirement: Locale selection is persisted locally
The `machado-web` application SHALL persist a manual locale selection in the browser so that subsequent visits preserve the user's preference.

#### Scenario: Manual selection is saved
- **WHEN** the user selects a supported locale manually
- **THEN** the application MUST store that locale in a local browser persistence mechanism controlled by the frontend

#### Scenario: Reload preserves selected locale
- **WHEN** the user reloads the page or starts a new session in the same browser after manually selecting a locale
- **THEN** the application MUST restore the previously selected locale before considering browser locale detection

### Requirement: Fixed interface text uses translation resources
The `machado-web` application SHALL render fixed interface text from translation resources instead of leaving visible hardcoded strings in prioritized user flows.

#### Scenario: Navigation and shared states are translated
- **WHEN** the application renders navbar items, shared buttons, loading states, empty states, error states, or pagination text
- **THEN** those fixed UI strings MUST be read from translation keys

#### Scenario: Auth and primary feature pages are translated
- **WHEN** the application renders login, register, home, list, and detail flows covered by this change
- **THEN** fixed titles, labels, helper messages, and action text MUST be read from translation keys

#### Scenario: Dynamic backend data is not translated by the frontend
- **WHEN** the application renders names or domain values returned directly by `machado-api`
- **THEN** the frontend MUST display those values as received unless an existing project pattern already defines a frontend translation for that specific datum

### Requirement: Missing translations fail gracefully
The `machado-web` application SHALL degrade safely when a translation key or locale resource is incomplete.

#### Scenario: Missing key uses fallback behavior
- **WHEN** a translation key is missing in the active locale resource
- **THEN** the application MUST use configured fallback behavior that avoids rendering a blank UI string

#### Scenario: Incomplete locale does not block rendering
- **WHEN** one locale resource is incomplete or temporarily behind another locale resource
- **THEN** the application MUST continue rendering the page using fallback translations where available

### Requirement: Language selector is accessible
The `machado-web` application SHALL keep the language selector operable and understandable for keyboard and assistive technology users.

#### Scenario: Selector exposes accessible naming
- **WHEN** the language selector is rendered in the navbar
- **THEN** it MUST provide an accessible label or equivalent semantic naming for assistive technologies

#### Scenario: Selector supports keyboard interaction
- **WHEN** a keyboard user navigates to the language selector
- **THEN** the user MUST be able to focus the control, inspect options, and change the locale without relying on pointer interaction only
