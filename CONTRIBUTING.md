# Contributing to CIRCE

Thank you for your interest in contributing to the CIRCE Agent Architecture Framework.

## How to Contribute

### Reporting Issues

Open a GitHub issue with:
- A clear description of the problem or suggestion
- Steps to reproduce (if applicable)
- Expected vs actual behavior
- Relevant file paths or YAML snippets

### Adding a New Skill

1. Create a folder under `.github/skills/<skill-name>/`
2. Add a `SKILL.md` following the existing skill format (see any skill in `.github/skills/` for reference)
3. Include templates in the skill folder if the skill generates YAML
4. Add the skill to the mapping table in `.github/copilot-instructions.md` if it should be user-invocable
5. Validate that the skill works end-to-end with the Author agent

### Adding a Domain Extension Pack

1. Create skills prefixed with the domain abbreviation (e.g., `bc-` for Business Central)
2. Include at minimum: setup skill, instruction patterns, action templates, and a blueprint
3. Add supporting documentation in `docs/`
4. Update the README to reference the new pack

### Modifying Conventions

Changes to `.github/circe-conventions.md` affect all agents and operations. Propose convention changes via a GitHub issue first to discuss impact.

## Development Rules

- **Follow the skill-first rule** — use existing skills whenever possible
- **Validate all YAML** — run `node .github/scripts/schema-lookup.bundle.js validate <file>` after edits
- **Write in Spanish** — all user-facing content (trigger phrases, messages, labels)
- **Document decisions** — create a decision record for significant design choices
- **Test before pushing** — use the Test agent or point-test via `chat-with-agent`

## Code of Conduct

Be respectful, constructive, and inclusive. This project follows the [Contributor Covenant](https://www.contributor-covenant.org/) code of conduct.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
