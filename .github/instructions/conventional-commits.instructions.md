# Conventional Commits

## Overview

All commit messages in this repository must follow the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/).

## Commit Message Format

Every commit message MUST be structured as follows:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Type

The type MUST be one of the following:

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files
- **revert**: Reverts a previous commit

### Scope

The scope is OPTIONAL and provides additional contextual information. It should be a noun describing a section of the codebase enclosed in parentheses.

Examples: `feat(parser)`, `fix(ui)`, `docs(readme)`

### Description

The description is REQUIRED and must:
- Use the imperative, present tense: "change" not "changed" nor "changes"
- Not capitalize the first letter
- Not include a period (.) at the end
- Be concise and clear about what the commit does

### Body

The body is OPTIONAL and should:
- Provide additional context about the change
- Explain the motivation for the change
- Contrast with previous behavior

### Footer

The footer is OPTIONAL and should contain:
- **BREAKING CHANGE**: A description of breaking changes (also indicated by `!` after type/scope)
- Issue references: e.g., `Closes #123`, `Fixes #456`

## Examples

### Simple commits
```
feat: add user authentication
fix: correct calculation in total score
docs: update installation instructions
style: format code with prettier
refactor: extract validation logic to separate function
```

### With scope
```
feat(leaderboard): add player name input
fix(timer): prevent negative time display
docs(readme): add development setup section
perf(game): optimize question generation algorithm
```

### With breaking change
```
feat!: change scoring system to linear time bonus

BREAKING CHANGE: The scoring system now uses linear time bonuses instead of fixed thresholds. 
Previous save files are incompatible.
```

### With body and footer
```
fix(audio): prevent sound overlap on rapid answers

Previously, clicking rapidly would cause sounds to overlap. Now we reset
the audio currentTime before playing to ensure clean sound playback.

Fixes #42
```

## Instructions for GitHub Copilot

When creating commit messages:

1. **Always** use conventional commit format
2. **Always** use imperative mood in the description
3. **Choose** the most appropriate type for the change
4. **Include** a scope when it adds clarity
5. **Write** clear, concise descriptions
6. **Add** a body if the change needs explanation
7. **Mark** breaking changes with `!` and BREAKING CHANGE footer
8. **Reference** issues when applicable

## Validation Checklist

Before committing, verify:
- [ ] Type is one of the allowed values
- [ ] Description uses imperative mood
- [ ] Description is lowercase (except for proper nouns)
- [ ] Description has no period at the end
- [ ] Breaking changes are marked with `!` and have BREAKING CHANGE footer
- [ ] Commit message is clear and descriptive
