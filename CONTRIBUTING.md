# Contributing to regulation-ledger

Thank you for your interest in contributing. This document explains how to set up your development environment and how to submit changes.

## Code of conduct

All contributors must follow our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold a respectful and inclusive environment.

## Development setup

### Prerequisites

- Node.js >= 20.0.0
- npm >= 10.0.0
- Git

### Clone and install

```bash
git clone https://github.com/clicsoluciones/regulation-ledger.git
cd regulation-ledger
npm install
```

### Run tests

```bash
npm run test
```

### Build

```bash
npm run build
```

### Lint

```bash
npm run lint
```

## Project structure

```
regulation-ledger/
├── packages/
│   ├── core/         # evidence-core: hash chain engine
│   ├── snapshot/     # regulation-snapshot: legal constants registry
│   ├── validate/     # compliance-validate: rules engine
│   └── api/          # ledger-api: HTTP API
├── docs/             # Architecture, roadmap, design docs
├── examples/         # Example use cases
└── .github/          # Issue templates, CI workflows
```

Each package in `packages/` is a standalone npm package that can be used independently. The workspace is a monorepo managed by npm workspaces.

## Submitting changes

### Pull request process

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Make your changes.
4. Add tests for new functionality.
5. Ensure `npm run test` and `npm run lint` pass.
6. Update documentation if you change public APIs.
7. Commit with a descriptive message following [Conventional Commits](https://www.conventionalcommits.org/).
8. Push to your fork.
9. Open a pull request against `main`.

### Commit message format

```
type(scope): short description

Longer description if needed.

Closes #issue_number
```

Types: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`, `perf`.

Examples:
- `feat(core): add Ed25519 signature support`
- `fix(snapshot): handle timezone correctly in getConstantsForDate`
- `docs(roadmap): update H4 deliverable list`

### Code style

- TypeScript strict mode
- 2-space indentation
- Single quotes for strings
- Semicolons required
- Run `npm run lint` before committing

## Reporting bugs

Open an issue using the bug report template. Include:
- Reproduction steps
- Expected behavior
- Actual behavior
- Node.js and npm versions
- Operating system

## Suggesting features

Open an issue using the feature request template. Describe:
- The problem you're trying to solve
- Your proposed solution
- Alternatives you've considered
- Whether you'd like to implement it yourself

## Security vulnerabilities

Do not open public issues for security vulnerabilities. See [SECURITY.md](SECURITY.md) for disclosure process.

## License

By contributing, you agree that your contributions will be licensed under the [Mozilla Public License 2.0](LICENSE).
