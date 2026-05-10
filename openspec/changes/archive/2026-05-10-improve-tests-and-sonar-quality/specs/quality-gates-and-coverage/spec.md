## ADDED Requirements

### Requirement: Web unit coverage gate
The `machado-web` project SHALL provide a reproducible unit test coverage gate for source files measured by Jest.

#### Scenario: Web unit coverage reaches target
- **WHEN** the maintainer runs the documented web unit coverage command in `machado-web`
- **THEN** the command MUST complete successfully and report 100% coverage for the measured statements, branches, functions and lines

#### Scenario: Web unit tests validate behavior
- **WHEN** new or adjusted Jest tests are added for components, hooks, services, route handlers, server actions or utilities
- **THEN** the tests MUST assert observable behavior, returned data, rendered state, validation errors or dependency calls instead of only importing modules

### Requirement: Web e2e integration coverage gate
The `machado-web` project SHALL provide reproducible Playwright e2e/integration coverage for the supported authenticated and public user flows.

#### Scenario: Web e2e suite succeeds
- **WHEN** the maintainer runs the documented Playwright command in `machado-web`
- **THEN** the command MUST complete successfully for the configured desktop and mobile projects

#### Scenario: Web e2e tests cover meaningful flows
- **WHEN** e2e/integration tests are added or adjusted
- **THEN** the tests MUST exercise user-visible flows through pages, navigation, route handlers or mocked backend interactions without introducing new product features

### Requirement: API unit coverage gate
The `machado-api` project SHALL provide a reproducible unit test coverage gate for source files measured by Pytest coverage.

#### Scenario: API coverage reaches target
- **WHEN** the maintainer runs `make test` in `machado-api`
- **THEN** the command MUST complete successfully and report 100% coverage for the measured API source scope

#### Scenario: Pokemon schema coverage is closed
- **WHEN** API tests are executed with coverage
- **THEN** `machado-api/app/domain/pokemon/schema.py` MUST be covered by tests for its validation, serialization and default behavior where applicable

### Requirement: Local SonarQube analysis
The monorepo SHALL provide or document a local SonarQube/SonarScanner setup that analyzes maintained API and Web source code.

#### Scenario: Local Sonar tooling is absent
- **WHEN** SonarQube or SonarScanner is not available in the maintainer environment
- **THEN** the implementation MUST provide a reproducible local installation or provisioning path without committing secrets

#### Scenario: Local Sonar analysis runs
- **WHEN** the maintainer runs the documented local SonarQube analysis command with required environment variables configured
- **THEN** the analysis MUST complete without requiring committed secrets and MUST include both `machado-api` and `machado-web` source and test coverage reports where available

#### Scenario: Generated files are excluded from Sonar
- **WHEN** SonarQube analyzes the monorepo
- **THEN** generated artifacts, dependency folders, coverage outputs, build outputs, caches, Playwright artifacts and Alembic migrations MUST be excluded from issue and coverage analysis

### Requirement: Sonar usage manual
The monorepo SHALL include a practical manual that teaches maintainers how to use SonarQube/SonarScanner in this project.

#### Scenario: Maintainer follows the Sonar manual
- **WHEN** a maintainer reads the Sonar usage manual
- **THEN** the manual MUST explain prerequisites, local setup, required environment variables, coverage generation, scanner execution, result inspection and basic troubleshooting without exposing real secrets

#### Scenario: Manual documents issue handling
- **WHEN** a maintainer reviews SonarQube findings using the manual
- **THEN** the manual MUST explain how to prioritize bugs, vulnerabilities and code smells, and how to document justified false positives or narrow suppressions

### Requirement: Sonar issues are resolved without behavior changes
The codebase SHALL resolve applicable SonarQube bugs, vulnerabilities and code smells in maintained source files without changing functional behavior.

#### Scenario: Applicable Sonar issues are fixed
- **WHEN** SonarQube reports issues in maintained API or Web source files
- **THEN** the implementation MUST fix the issues through tests, small refactors, typing, simplification or bug fixes while preserving existing contracts

#### Scenario: False positives are justified
- **WHEN** a SonarQube issue is a false positive or cannot be changed safely without altering intended behavior
- **THEN** the implementation MUST document the justification and limit any suppression to the narrowest affected code or configuration

### Requirement: Existing quality commands remain green
The project SHALL preserve the existing lint, build and test validation commands for API and Web.

#### Scenario: API quality commands pass
- **WHEN** the maintainer runs `make lint` and `make test` in `machado-api`
- **THEN** both commands MUST complete successfully

#### Scenario: Web quality commands pass
- **WHEN** the maintainer runs `yarn lint`, `yarn build` and `yarn test` in `machado-web`
- **THEN** all commands MUST complete successfully
