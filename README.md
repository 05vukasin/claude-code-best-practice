# Claude Code Best Practices Guide

A comprehensive, beautifully designed reference guide for setting up and using [Claude Code](https://claude.ai) effectively. This single-page website covers everything from project structure to advanced features like subagents, hooks, and MCP servers.

## Preview

The guide is a fully responsive, single-file HTML page with:

- Dark / Light theme toggle
- Serbian / English language switcher
- Sidebar navigation with scroll-based active highlighting
- Mobile-friendly responsive layout
- Code examples, tables, checklists, and interactive accordions

## What's Covered

| Section | Topics |
|---------|--------|
| **1. Project Structure** | `.claude/` directory layout, user-level vs project-level config |
| **2. CLAUDE.md** | Placement hierarchy, what to include/exclude, importing files |
| **3. Settings** | Permission scopes, `settings.json` examples, priority hierarchy |
| **4. Skills** | Custom slash commands, SKILL.md format, arguments, dynamic context |
| **5. Subagents** | Built-in agents, creating custom agents, invocation methods |
| **6. Hooks** | Lifecycle events, exit codes, file protection, auto-formatting |
| **7. MCP Servers** | stdio/http/sse/ws server types, configuration |
| **8. Rules** | Modular instructions, path-specific rules |
| **9. Memory** | Auto memory types, storage location, file format |
| **10. Checklist** | Setup checklist for new projects, file path quick reference |

## Getting Started

1. Clone the repository:
   ```bash
   git clone https://github.com/05vukasin/claude-code-best-practice.git
   ```

2. Open `index.html` in your browser -- no build step required.

## Features

### Bilingual Support (English / Serbian)

Click the **SRB** / **ENG** button in the sidebar footer to switch between English and Serbian. Your preference is saved in localStorage.

### Dark Mode

Click **Toggle Theme** to switch between light and dark modes. Respects your system preference on first visit.

### Responsive Design

Works on desktop, tablet, and mobile. The sidebar collapses into a hamburger menu on smaller screens.

## Tech Stack

- Pure HTML, CSS, and vanilla JavaScript
- No dependencies, no build tools, no frameworks
- Google Fonts (Inter + JetBrains Mono)

## Source Content

The guide content is based on `claude-best-practices.md`, a comprehensive markdown reference covering all Claude Code configuration options.

## Contributing

Contributions are welcome! Feel free to open issues or submit pull requests for:

- Additional translations
- Content updates as Claude Code evolves
- Design improvements
- Accessibility enhancements

## License

This project is licensed under the MIT License -- see the [LICENSE](LICENSE) file for details.
