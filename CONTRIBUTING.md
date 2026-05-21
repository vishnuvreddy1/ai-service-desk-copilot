# Contributing to AI Service Desk Copilot

Thank you for your interest in contributing! This project welcomes improvements to documentation, flow designs, topic definitions, and architecture patterns.

---

## How to Contribute

### 1. Fork the Repository
Click **Fork** at the top right of the repo page to create your own copy.

### 2. Create a Branch
```bash
git checkout -b feature/your-feature-name
```

Use a descriptive branch name:
- `feature/add-sentiment-analysis-topic`
- `fix/escalation-flow-error-handling`
- `docs/update-deployment-guide`

### 3. Make Your Changes
- Follow the existing folder structure
- Update the relevant `README.md` in the affected folder
- If adding a new Copilot Studio topic, document it in `copilot-studio/topics/README.md`
- If adding a new Power Automate flow, create a new subfolder under `power-automate/` with a `README.md`

### 4. Commit Your Changes
```bash
git add .
git commit -m "feat: add sentiment analysis escalation trigger"
```

**Commit message format:**
- `feat:` — new feature or topic
- `fix:` — bug fix or correction
- `docs:` — documentation only changes
- `refactor:` — restructuring without functional change
- `chore:` — maintenance tasks

### 5. Push and Open a Pull Request
```bash
git push origin feature/your-feature-name
```
Then open a Pull Request against the `main` branch with:
- A clear title describing the change
- A description of what was changed and why
- Screenshots or flow diagrams if applicable

---

## Contribution Ideas

Here are areas where contributions are especially welcome:

| Area | Ideas |
|---|---|
| **New Topics** | Multi-language support, VPN issues, printer setup |
| **Flow Improvements** | Better retry logic, dead-letter queue handling |
| **Security** | Conditional access policies, PIM integration |
| **Analytics** | Power BI report templates for bot analytics |
| **Documentation** | Architecture diagrams, video walkthroughs |
| **Testing** | UAT test scripts, load testing patterns |

---

## Code of Conduct

- Be respectful and constructive in all interactions
- Focus feedback on the work, not the person
- Keep discussions relevant to the project

---

## Questions?

Open a [GitHub Issue](../../issues) with the `question` label, or reach out via [LinkedIn](https://linkedin.com/in/vishnu-reddy-v-403b2934b).
