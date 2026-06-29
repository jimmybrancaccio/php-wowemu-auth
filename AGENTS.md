# AGENTS.md

## Project Overview
This repository is a small Composer library that implements WoW emulator compatible SRP6 authentication helpers for CMaNGOS-style account registration and login flows. It is written in PHP, exposes a PSR-4 API under `Laizerox\Wowemu\SRP`, and uses phpseclib plus `ext-bcmath` for big integer cryptographic operations.

## Repository Structure
```text
.
├── src/
│   └── Wowemu/SRP/
│       ├── Client.php       # Shared SRP state, proofs, random values, and base behavior.
│       ├── UserClient.php   # Client-side registration/login computations.
│       └── HostClient.php   # Host/server-side session computation and proof validation.
├── tests/
│   ├── UserClientTest.php           # Salt, verifier, and input validation coverage.
│   └── SRPClientIntegrationTest.php # End-to-end user/host SRP handshake coverage.
├── .github/
│   ├── workflows/php.yml            # CI matrix for PHP and operating systems.
│   └── copilot-instructions.md      # Existing agent guidance for this repo.
├── composer.json            # Composer package metadata, dependencies, autoloading, and scripts.
├── composer.lock            # Locked dependency graph for local/CI installs.
├── phpunit.xml              # PHPUnit suite configuration.
├── README.md                # Public installation and usage examples.
└── LICENSE                  # GPL-3.0-or-later license text.
```

## Environment Setup
1. Use PHP 8.2 through 8.5 with the `bcmath` extension enabled.
2. Install Composer dependencies:
   ```bash
   composer install
   ```
3. No application runtime or `.env` file is required for the library test suite.
4. For real usage, consumers provide their own web app, Composer autoloader, and CMaNGOS database connection as shown in `README.md`.

## Build & Run
This is a library, so there is no build step or long-running development server.

Useful local commands:
```bash
composer validate
composer install
composer run-script test
```

Production usage is through Composer package installation and PHP autoloading:
```bash
composer require laizerox/php-wowemu-auth
```

## Testing
Tests use PHPUnit and live in `tests/`. Run the full suite with:
```bash
composer run-script test
```

The direct equivalent is:
```bash
vendor/bin/phpunit
```

CI runs `composer validate`, resolves dependencies with `composer update --prefer-dist --no-progress --no-interaction`, and runs `composer run-script test` across PHP 8.2, 8.3, 8.4, and 8.5 on Ubuntu, Windows, and macOS.

## Code Style & Conventions
- Follow the existing PHP style in touched files; do not reformat unrelated code.
- Preserve the `Laizerox\Wowemu\SRP` namespace and PSR-4 mapping from `src/`.
- Keep public method names and return shapes stable unless an API break is explicitly requested.
- Use PHPUnit attribute-based tests such as `#[DataProvider]`, matching the current test style.
- Run `composer run-script test` before handing off behavior changes.
- In interactive `zsh`, prefer `composer run-script test` over `composer test` to avoid shell autocorrect prompts.
- No separate formatter or linter is configured in this repository; use `composer validate` and PHPUnit as the current automated checks.
- Run shell commands with `/bin/bash` when acting as an agent in this repo.

## Architecture Notes
- `Client` is the shared base for SRP parameters, ephemeral values, session proofs, and random byte generation.
- `UserClient` owns registration/login-side behavior: salt generation, private key computation, verifier generation, client public ephemeral value, session key calculation, and host proof validation.
- `HostClient` owns server-side behavior: verifier storage input, host public ephemeral value, session key calculation, and client proof validation.
- SRP protocol changes should be made carefully and covered by `tests/SRPClientIntegrationTest.php`, which documents the intended client/host message order.
- Cryptographic big integer operations go through `phpseclib\Math\BigInteger`; do not replace this with ad hoc integer/string math.
- The library does not manage database access, sessions, HTTP requests, or user validation. Those responsibilities belong to consuming applications.

## Commit & PR Conventions
- No explicit branch naming or commit message convention is defined in the repository.
- Keep commits small and focused, especially for protocol or public API changes.
- PRs should mention the PHP versions affected, summarize any public API changes, and include the result of `composer run-script test`.
- For SRP math changes, note whether verifier generation, client proof validation, host proof validation, or the full handshake was updated.

## Things to Avoid
- Do not introduce framework-specific application code, controllers, database queries, or session handling into this library.
- Do not change SRP constants, hash ordering, byte order, salt handling, verifier logic, or proof formulas without focused tests.
- Do not bypass `random_bytes()` for security-sensitive random values.
- Do not remove `ext-bcmath` or phpseclib requirements without replacing all dependent behavior safely.
- Do not break consumers by renaming the `Laizerox\Wowemu\SRP` classes or changing public method signatures casually.
- Do not commit generated dependency or test cache directories such as `vendor/` or `.phpunit.cache/`.
