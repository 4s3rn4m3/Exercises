# Conventional Commits

To improve your commits and ensure they cover everything, the secret lies not only in the prefix but in the full structure of the message and the mindset you use to divide your work. The Conventional Commits standard is very powerful, but to make it truly useful, you should leverage the body and footer of the message, as well as expand the available types.

To organize the types of changes, you can use this reference:

| Type | Purpose | Example |
| :--- | :--- | :--- |
| `feat` | A new feature for the user. | `feat(auth): add google oauth2 login` |
| `fix` | Bug fix. | `fix(ui): resolve overlap in mobile menu` |
| `docs` | Changes only to documentation. | `docs(readme): add installation steps` |
| `style` | Visual or formatting changes that do not affect logic (whitespace, commas). | `style(css): update primary button padding` |
| `refactor` | Code change that neither fixes a bug nor adds a feature. | `refactor(db): optimize user query` |
| `perf` | Code change to improve performance. | `perf(images): implement lazy loading` |
| `test` | Adding missing tests or correcting existing ones. | `test(api): add unit tests for user controller` |
| `build` | Changes that affect the build system or external dependencies. | `build(npm): update react to v18` |
| `ci` | Changes to CI configuration files and scripts (GitHub Actions, Jenkins). | `ci(github): add lint check to pr workflow` |
| `chore` | Routine tasks that do not modify production code. | `chore(git): add .gitignore for vscode` |
| `revert` | Revert a previous commit. | `revert: undo feat(api): add user auth` |

The golden rule is to make **atomic commits**: one commit should do one thing. If you fixed a bug and also happened to change a color and update the README, don't use a single commit with an ambiguous prefix; make three separate commits.

For more complex messages, use the full three-part structure:

### Commit Structure

The header is what you already know: `<type>(<scope>): <short description>`. The description should be written in the **imperative mood** (e.g., "add" instead of "added" or "adds"), as if you were giving an order to the code.

The body is an optional paragraph separated from the header by a blank line. This is where you explain the **why** of the change and not the **how**. The "how" is already visible in the code; the "why" is what the team needs to know months later to understand the technical decision.

The footer is the final section where ticket references or breaking changes are noted. If a change breaks backward compatibility, you must start the footer with `BREAKING CHANGE:`, followed by a description of the change and how to migrate. It is also the ideal place to close issues using keywords like `Closes #123` or `Fixes #456`.
