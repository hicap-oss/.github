# Contributing to Hicap Projects

Thank you for your interest in contributing to Hicap! We welcome contributions from the community and are grateful for your support.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Process](#development-process)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Reporting Bugs](#reporting-bugs)
- [Requesting Features](#requesting-features)
- [Questions](#questions)

## Code of Conduct

This project and everyone participating in it is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## Getting Started

1. **Fork the repository** - Create your own fork of the project
2. **Clone your fork** - Clone the repository to your local machine
3. **Create a branch** - Create a new branch for your changes
4. **Make your changes** - Implement your bug fix or feature
5. **Test your changes** - Ensure all tests pass
6. **Submit a pull request** - Open a PR with a clear description

## How to Contribute

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- A clear and descriptive title
- Detailed steps to reproduce the issue
- Expected vs actual behavior
- Screenshots or error messages (if applicable)
- Environment details (OS, version, etc.)

Use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.yml) when available.

### Requesting Features

Feature requests are welcome! When submitting a feature request:

- Use a clear and descriptive title
- Provide a detailed description of the proposed feature
- Explain why this feature would be useful
- Include examples of how it would work

Use the [feature request template](.github/ISSUE_TEMPLATE/feature_request.yml) when available.

### Contributing Code

1. **Check existing issues** - Look for existing issues or create a new one
2. **Discuss major changes** - For significant changes, discuss with maintainers first
3. **Follow coding standards** - Adhere to the project's coding style
4. **Write tests** - Include tests for new features
5. **Update documentation** - Keep docs in sync with code changes

## Development Process

### Setting Up Your Development Environment

```bash
# Clone your fork
git clone https://github.com/YOUR-USERNAME/REPOSITORY-NAME.git
cd REPOSITORY-NAME

# Add upstream remote
git remote add upstream https://github.com/hicap-oss/REPOSITORY-NAME.git

# Create a new branch
git checkout -b feature/your-feature-name
```

### Running Tests

```bash
# Run the test suite (adjust based on project)
npm test          # For Node.js projects
pytest            # For Python projects
./gradlew test    # For Java projects
```

### Keeping Your Fork Updated

```bash
# Fetch upstream changes
git fetch upstream

# Merge upstream changes into your branch
git checkout main
git merge upstream/main
```

## Coding Standards

- Write clear, readable, and maintainable code
- Follow the existing code style in the project
- Use meaningful variable and function names
- Comment complex logic and non-obvious code
- Keep functions small and focused
- Write self-documenting code where possible

### Language-Specific Guidelines

- **JavaScript/TypeScript**: Follow ESLint/Prettier configurations
- **Python**: Follow PEP 8 style guide
- **Java**: Follow Google Java Style Guide
- **Go**: Follow official Go formatting standards

## Commit Guidelines

We follow [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Commit Types

- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `perf`: Performance improvements
- `ci`: CI/CD changes

### Examples

```
feat(auth): add OAuth2 authentication support

Implemented OAuth2 authentication flow with support for
multiple providers including Google and GitHub.

Closes #123
```

```
fix(api): resolve race condition in user creation

Added proper locking mechanism to prevent duplicate
user creation when concurrent requests occur.

Fixes #456
```

## Pull Request Process

1. **Update documentation** - Ensure README and other docs are updated
2. **Add tests** - Include tests for new functionality
3. **Update changelog** - Add entry to CHANGELOG.md (if applicable)
4. **Ensure CI passes** - All automated checks must pass
5. **Request review** - Tag appropriate reviewers
6. **Address feedback** - Respond to review comments promptly
7. **Squash commits** - Clean up commit history before merging (if requested)

### Pull Request Title

Use a clear, descriptive title following the commit convention:

```
feat: add user authentication module
fix: resolve memory leak in cache handler
docs: update API documentation for v2.0
```

### Pull Request Description

Include:
- Summary of changes
- Motivation and context
- Related issue numbers
- Screenshots (for UI changes)
- Testing performed
- Breaking changes (if any)

## Code Review

- Be respectful and constructive
- Explain the reasoning behind suggestions
- Accept that there may be multiple valid solutions
- Focus on the code, not the person
- Respond to feedback in a timely manner

## Release Process

Releases are managed by project maintainers. The process typically includes:

1. Version bump according to [Semantic Versioning](https://semver.org/)
2. Update CHANGELOG.md
3. Create a release tag
4. Publish release notes

## License

By contributing to Hicap projects, you agree that your contributions will be licensed under the same license as the project (check the LICENSE file in each repository).

## Recognition

We value all contributions and maintain a list of contributors in our repositories. Your contributions will be recognized in:

- Repository contributor list
- Release notes (for significant contributions)
- Project documentation (where applicable)

## Getting Help

If you need help or have questions:

- Check the [SUPPORT.md](SUPPORT.md) file
- Review existing documentation
- Search through existing issues
- Join our community discussions
- Contact maintainers directly for sensitive matters

## Additional Resources

- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
- [Code Review Best Practices](https://google.github.io/eng-practices/review/)

Thank you for contributing to Hicap! 🎉
