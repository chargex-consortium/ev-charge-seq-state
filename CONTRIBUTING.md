# Contributing to EVChargeSeqState

First off, thank you for your interest in improving and expanding this repository! We welcome contributions of all kinds-from typo fixes and documentation improvements to new diagrams and template enhancements.

## 📋 How to Report Issues

1. **Search for existing issues**: Before opening a new issue, check to see if someone else has already reported the same problem.
2. **Use clear titles**: Summarize the issue in a concise, descriptive title.
3. **Provide details**: Include steps to reproduce, expected vs. actual behavior, and any relevant files or logs.
4. **Labels**: Add labels like `bug`, `enhancement`, or `question` to help maintainers triage.

## 💡 Suggesting Enhancements

- Open an issue with title prefixed by `enhancement:` (e.g., `enhancement: add ISO 15118-20 sequence`).
- Describe why the feature or update is needed and any design ideas.
- If possible, include a rough sketch or PlantUML source of the proposed diagram.

## 🛠 Pull Request Process

1. **Fork** the repository to your personal account.  
2. **Create a branch**: Use a descriptive name, e.g. `feature/hlc-schedule-format` or `bugfix/readme-typo`.  
3. **Commit changes**: Write clear, atomic commits. Use the imperative mood in commit messages (e.g., `Add README for pwm-charging`).  
5. **Verify**: Check that:
   - SVG diagrams display correctly in your local browser.  
   - Each diagram folder has a `README.md` (or `NOTES.md`) with a brief description and embedded image link.  
   - The `README.md` is updated if you added new protocols or actors.
6. **Push** your branch to GitHub.  
7. **Open a Pull Request** against `main`.

## 🎨 Diagram & File Standards

- **PlantUML source (`.puml`)**: Place in the appropriate subfolder under `diagrams/sequence/` or `diagrams/state-machine/`.
- **Rendered images**: Export as `.svg` and `.png` with the same base filename.  
- **Documentation**: Each diagram folder must include a Markdown file (`README.md`) that:
  - Embeds the diagram at the top.
  - Summarizes key steps or states.
  - Links back to the PlantUML source.
- **Naming**: Keep names lowercase and hyphen-separated (e.g., `hlc-schedule.puml`).

## 📄 License

By contributing, you agree that your contributions will be licensed under the project’s [Apache 2.0 License](LICENSE).

---



