# Server App - Central Documentation

[![Documentation](https://img.shields.io/badge/docs-mkdocs-blue)](https://miltoneo.github.io/server-app-docs/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/status-active-success)]()

> Central documentation hub for the server-app multi-repository ecosystem

## 🎯 About This Repository

This repository contains the **centralized documentation** for all server-app projects. It serves as the single source of truth for:

- 🏗️ **Architecture** - System design and architectural decisions
- 📋 **Standards** - Coding standards and conventions
- 🔄 **Processes** - Development workflows and procedures
- 📚 **Guides** - How-to guides and tutorials
- 🔗 **Project Catalog** - Overview of all 8+ projects in the ecosystem

## 📊 Ecosystem Overview

### Active Projects

| Project | Description | Repository | Status |
|---------|-------------|------------|--------|
| **Construtora** | Construction project management | [server-app-construtora](https://github.com/Miltoneo/server-app-construtora) | ✅ Active |
| **Medicos** | Medical financial management | [server-app-medicos](https://github.com/Miltoneo/server-app-medicos) | ✅ Active |
| **Emprestimos** | Loan management system | [server-app-emprestimos](https://github.com/Miltoneo/server-app-emprestimos) | ✅ Active |
| **SISU** | SISU grades consultation | [server-app-sisu](https://github.com/Miltoneo/server-app-sisu) | ✅ Active |
| **Mais Médicos** | Mais Médicos program management | [server-app-maismedicos](https://github.com/Miltoneo/server-app-maismedicos) | ✅ Active |
| **Rotinas** | Routines and procedures | [server-app-rotinas](https://github.com/Miltoneo/server-app-rotinas) | ✅ Active |
| **Eleição** | Electoral data analysis | [server-app-eleicao](https://github.com/Miltoneo/server-app-eleicao) | ✅ Active |
| **TDS** | IoT telemetry (TimescaleDB) | [server-app-tds](https://github.com/Miltoneo/server-app-tds) | ✅ Active |

### Shared Repositories

| Repository | Description | Link |
|------------|-------------|------|
| **Shared** | Shared Python library | [server-app-shared](https://github.com/Miltoneo/server-app-shared) |
| **Infrastructure** | DevOps and infrastructure | [server-app-infrastructure](https://github.com/Miltoneo/server-app-infrastructure) |
| **Original Monorepo** | Archived monorepo (read-only) | [server-app](https://github.com/Miltoneo/server-app) |

## 📚 Documentation Structure

```
docs/
├── architecture/          # System architecture and design
│   ├── overview.md        # Ecosystem overview
│   ├── multi-repo.md      # Multi-repo strategy
│   └── data-flow.md       # Data flow between systems
│
├── standards/             # Development standards
│   ├── coding.md          # Coding standards
│   ├── git-workflow.md    # Git workflow (GitFlow)
│   └── api-design.md      # REST API design
│
├── processes/             # Work processes
│   ├── onboarding.md      # Developer onboarding
│   ├── deployment.md      # Deployment process
│   └── code-review.md     # Code review guidelines
│
├── guides/                # Practical guides
│   ├── getting-started.md # Quick start guide
│   ├── local-setup.md     # Local development setup
│   └── troubleshooting.md # Common issues
│
├── adr/                   # Architecture Decision Records
│   ├── 0001-multi-repo.md # ADR: Multi-repo strategy
│   └── template.md        # ADR template
│
├── migration/             # Migration documentation
│   ├── overview.md        # Migration overview
│   ├── phase-0.md         # Phase 0: Preparation
│   └── phase-4.md         # Phase 4: Archival
│
└── projects/              # Project-specific docs
    ├── construtora.md     # Construtora project
    ├── medicos.md         # Medicos project
    └── ...                # Other projects
```

## 🚀 Quick Start

### For New Developers

1. **Read the overview**: Start with [Getting Started Guide](docs/guides/getting-started.md)
2. **Setup local environment**: Follow [Local Setup Guide](docs/guides/local-setup.md)
3. **Understand architecture**: Read [Architecture Overview](docs/architecture/overview.md)
4. **Choose a project**: See [Projects Catalog](docs/projects/overview.md)

### For Existing Team Members

- 📋 [Standards](docs/standards/) - Review coding standards
- 🔄 [Processes](docs/processes/) - Understand workflows
- 🔍 [Troubleshooting](docs/guides/troubleshooting.md) - Common issues

## 🛠️ Contributing to Documentation

We welcome contributions! Here's how:

### 1. Clone the Repository

```bash
git clone https://github.com/Miltoneo/server-app-docs
cd server-app-docs
```

### 2. Install MkDocs

```bash
pip install mkdocs-material
pip install mkdocs-mermaid2-plugin
```

### 3. Run Local Server

```bash
mkdocs serve
# Open http://localhost:8000
```

### 4. Make Changes

Edit Markdown files in the `docs/` folder.

### 5. Submit Pull Request

```bash
git checkout -b docs/your-change
git add .
git commit -m "docs: your change description"
git push origin docs/your-change
# Create PR on GitHub
```

### Documentation Standards

- ✅ Use Markdown format
- ✅ Include frontmatter (title, date, author)
- ✅ Add relevant links and references
- ✅ Test locally before submitting
- ✅ Get at least 1 approval before merging

## 🔍 Search

The documentation site includes full-text search. Use the search bar in the navigation.

## 📝 Documentation Templates

Templates are available for common document types:

- [ADR Template](docs/adr/template.md) - Architecture Decision Records
- [Guide Template](docs/guides/template.md) - How-to guides
- [Process Template](docs/processes/template.md) - Process documentation

## 🔗 Related Resources

### External Documentation

- [Django Documentation](https://docs.djangoproject.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)

### Internal Links

- [Infrastructure Docs](https://github.com/Miltoneo/server-app-infrastructure)
- [Shared Library API](https://github.com/Miltoneo/server-app-shared)
- [Migration Documentation](docs/migration/overview.md)

## 📞 Support

Need help?

- 💬 **Slack**: #dev-docs
- 📧 **Email**: dev@onkoto.com
- 🐛 **Issues**: [GitHub Issues](https://github.com/Miltoneo/server-app-docs/issues)

## 📄 License

This documentation is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Last Updated:** January 9, 2026  
**Maintained by:** DevOps & Engineering Team  
**Documentation Site:** https://miltoneo.github.io/server-app-docs/
