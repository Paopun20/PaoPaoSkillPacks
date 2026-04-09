---
name: good-readme
description: Write a high-quality, professional README.md for a software project, following the opinionated best-practice guide by Bane Sullivan (banesullivan/README). Use this skill whenever the user asks to write, generate, improve, or review a README — even if they just say "write a README for my project", "help me document this repo", or "make my README better". Also trigger when the user pastes code and asks how to present or share it, or when they're preparing to open-source a project. Don't wait for the user to say "good README" — this skill applies any time a README is involved.
---

# Good README Skill

A guide for writing READMEs that make people actually want to use your software. Based on [banesullivan/README](https://github.com/banesullivan/README).

> Your README is often the first and only thing anyone will see about your software. People judge your software by your README. A poorly written README implies poorly written software.

---

## Core Philosophy

- **Write for a first-year CS student.** If they can understand it, anyone can.
- **Answer these 3 questions fast:** What does it do? How does it work? Who made it?
- **Sell it.** Your README is your best (and often only) marketing material.
- **Keep it concise.** Ain't nobody got time for a manifesto.
- **Link to everything** relevant: docs, CI badges, deployments, examples, related projects.

---

## README Structure (in order)

### 1. 🌟 Highlights *(most important section)*
A concise bulleted list of the software's main selling points. Lead with the best thing about it. Use emojis — they're friendly, quirky, and help navigation.

```markdown
## ✨ Highlights
- 🚀 Zero-config setup — works out of the box
- 📦 Ships as a single binary with no dependencies
- 🔌 Integrates with any CI pipeline
```

### 2. 📖 Overview
Two paragraphs max. Cover:
- What the software does and how it works (high level)
- Who made it and why it exists
- Optionally: how it fits into the broader ecosystem (respectful comparisons to alternatives are fine)

### 3. 🖼️ Demo / Screenshot
Show the software in action. A picture is worth a thousand words of documentation.
- Screenshots for GUIs
- One-liners for CLIs — keep them minimal, don't show full API detail
- GIFs or short videos if possible

### 4. 🔧 Installation
Step-by-step. Assume nothing. Copy-paste ready commands.

```markdown
pip install mypackage
```

### 5. 🚀 Usage / Quickstart
A minimal working example that solves a common problem. Show the most appealing use case first. Link to full docs for details — don't replicate them here.

### 6. 📚 Documentation / Links
If the project has dedicated docs, make this section a **link fest** — point to everything:
- Full docs site
- API reference
- Tutorials
- Changelog
- Contributing guide

> Once a project has a dedicated docs site, the README should become a minimal elevator pitch + link collection.

### 7. 🤝 Contributing *(optional)*
Brief instructions or link to CONTRIBUTING.md.

### 8. 👤 About the Author
A short subsection: who created this, why, what's their background. Humanizes the project.

### 9. 📄 License
One line, with a badge.

---

## Writing Tips

**Emoji in headers:** Use them. They aid scanning and make the doc feel approachable.

**Badges:** Add CI status, coverage, PyPI/npm version, license at the very top. They signal quality instantly.

**Tone:** Enthusiastic but not hyperbolic. Direct. Friendly. First-year CS student should feel welcomed.

**Length:** Shorter is better. If you have more to say, write docs and link to them.

**Comparisons:** If relevant, briefly and respectfully compare to alternatives. Helps users self-select.

**Template:** After writing, offer the user a blank template they can fill in for future projects.

---

## When to Use What Format

| Situation | README should be |
|---|---|
| Early project, no docs site yet | Full structure above |
| Project has dedicated docs site | Minimal: elevator pitch + links |
| CLI tool | Heavy on one-liner examples |
| Library/SDK | Heavy on quickstart code snippet |
| Open-source, seeking contributors | Add Contributing + About sections |

---

## Checklist Before Finishing

- [ ] Answers: what, how, who — in the first screenful
- [ ] Highlights section at the top
- [ ] At least one demo (screenshot, GIF, or code example)
- [ ] Copy-paste install command
- [ ] Links to all relevant resources
- [ ] Written for a non-expert reader
- [ ] Badges for CI / version / license
- [ ] No walls of text — use headers, bullets, code blocks
