# LLM Context: CakePHP 5.2 PT-BR Documentation Project

## Project Identity
**Type**: MkDocs Material documentation site
**Language**: Portuguese (Brazil)
**Source**: CakePHP 5.2 documentation (translated and expanded)
**Status**: Active development (34 pages completed)
**Quality**: Average 8.42/10 (13 excellent, 16 good, 3 needs improvement)

## File Structure
```
cakephp/                                    # Parent directory
├── .claude/
│   └── LLM_CONTEXT.md                     # This file (context for LLM agents)
└── documentacao-melhorada/                 # Project root - THIS IS THE WORKING DIRECTORY
    ├── mkdocs.yml                          # MkDocs Material config
    ├── requirements.txt                    # Python deps: mkdocs>=1.5.0, mkdocs-material>=9.5.0
    ├── README.md                           # Project overview
    ├── .venv/                              # Virtual environment (active)
    └── docs/                               # 34 markdown files
        ├── css/extra.css                  # Custom CakePHP branding
        ├── index.md                       # Homepage
        ├── CONTROLE_PROGRESSO.md          # Progress tracking (may not exist)
        └── RANKING_QUALIDADE.md           # Quality analysis (may not exist)
```

**Note**: All file paths in this document are relative to `documentacao-melhorada/` (the project working directory). When referencing files:
- Use `mkdocs.yml` or `./mkdocs.yml` for root files
- Use `docs/index.md` for documentation pages
- Use `docs/css/extra.css` for stylesheets

## Navigation Structure (./mkdocs.yml)
```yaml
nav:
  - Início: [index.md, quickstart.md, where-to-get-help.md]
  - Introdução: [intro.md, installation.md, conventions.md, cakephp-folder-structure.md]
  - Tutoriais: [tutorials-and-examples.md, CMS Tutorial (7 pages)]
  - Controllers: [controllers.md, request-response, middleware, components, pagination, pages]
  - Views: [views.md]
  - Core Libraries: [hash.md, inflector.md, number.md, httpclient.md, registry-objects.md, global-constants-and-functions.md]
  - Internacionalização: [internationalization-and-localization.md]
  - Segurança: [security.md]
  - Logging: [logging.md]
  - Plugins: [plugin.md]
  - Sobre: [CONTROLE_PROGRESSO.md, RANKING_QUALIDADE.md]
```
All paths above are relative to `docs/` directory.

## Theme Configuration
```yaml
theme:
  name: material
  language: pt-BR
  palette:
    primary: red          # CakePHP brand color
    accent: deep orange
  features:
    - navigation.tabs
    - navigation.tabs.sticky
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.suggest
    - search.highlight
    - content.tabs.link
    - content.code.copy
```

## Markdown Extensions Enabled
- `toc` (permalink, toc_depth: 3)
- `admonition` (callout boxes)
- `pymdownx.details` (collapsible sections)
- `pymdownx.superfences` (advanced code blocks)
- `pymdownx.tabbed` (content tabs with `=== "Tab Name"` syntax)
- `pymdownx.highlight` (syntax highlighting)
- `pymdownx.inlinehilite`
- `pymdownx.snippets`
- `tables`
- `attr_list`
- `md_in_html`

## Custom CSS (docs/css/extra.css)
**Variables**:
```css
:root {
  --cakephp-dark: #1B1B1B;
  --cakephp-gray: #F5F5F5;
  --success-green: #28a745;
  --warning-yellow: #ffc107;
  --info-blue: #17a2b8;
}
```

**Key Styles**: Code blocks, admonitions (info/tip/warning/danger), tables, headers with CakePHP red (#D33C44) borders, badges, responsive design

## Content Guidelines
1. **Expansion**: Average 300-400% from original EN docs
2. **Examples**: Complete, functional PHP code with Brazilian context (CPF, CNPJ, timezone America/Sao_Paulo)
3. **Diagrams**: Mermaid.js for MVC flow, request lifecycle, middleware stack
4. **Security**: Emphasis on XSS, CSRF, SQL injection prevention
5. **Admonitions**: Use `!!! info/tip/warning/danger "Title"` for callouts
6. **Tabs**: Use `=== "Tab Name"` for code variations (PHP 8.1 vs 8.0, different approaches)
7. **Translation Quality**: Natural PT-BR, not literal translation

## Quality Assessment (docs/RANKING_QUALIDADE.md - may not exist)
**Criteria** (0-10 scale):
- Completude (average: 8.81)
- Qualidade da Tradução (average: 9.72)
- Melhorias (average: 9.31)
- Formatação MkDocs (average: 5.83)

**Top Pages** (≥9.0):
docs/quickstart.md (9.58), docs/where-to-get-help.md (9.42), docs/conventions.md (9.33), docs/intro.md (9.25), docs/cakephp-folder-structure.md (9.17), docs/installation.md (9.17), docs/cms-authorization.md (9.08), docs/views.md (9.00)

**Priority Improvements** (<7.0):
1. docs/controllers-pagination.md (5.25) - only 16% complete, needs expansion
2. docs/controllers-components.md (5.50) - only 20% complete, missing examples
3. docs/controllers-request-response.md (5.88) - only 20% complete, lacks depth

## Common Commands
```bash
# All commands assume you're in documentacao-melhorada/ directory

# Activate environment
source .venv/bin/activate

# Install/update dependencies
pip install -r requirements.txt

# Development server
mkdocs serve  # http://127.0.0.1:8000/

# Production build
mkdocs build  # Output to site/

# Check structure
tree -L 2 -I 'site|__pycache__'
```

## Development Notes
- **Working directory**: `documentacao-melhorada/` (all commands run from here)
- **Virtual env**: `.venv/` in working directory (not `venv/`)
- **Dependencies**: All installed (mkdocs 1.6.1, mkdocs-material 9.7.0, pymdown-extensions 10.17.1)
- **Removed**: emoji extension (caused errors), --cakephp-red CSS variable (replaced with #D33C44)
- **This context file**: `../.claude/LLM_CONTEXT.md` (relative to working directory)
- **Original source**: CakePHP 5.2 official documentation (no longer in this directory structure)

## Page Status Summary
| Section | Pages | Status |
|---------|-------|--------|
| Início | 3 | ✅ Complete |
| Introdução | 4 | ✅ Complete |
| Tutoriais | 8 | ✅ Complete |
| Controllers | 6 | ⚠️ 3 need improvement |
| Views | 1 | ✅ Complete |
| Core Libraries | 6 | ✅ Complete |
| I18n/Security/Logging/Plugins | 4 | ✅ Complete |
| ORM | 0 | ❌ Not started |
| Development | 0 | ❌ Not started |
| Console | 0 | ❌ Not started |

## Content Patterns
**Admonition Examples**:
```markdown
!!! info "Informação"
    Texto informativo

!!! tip "Dica"
    Dica útil

!!! warning "Atenção"
    Aviso importante

!!! danger "Perigo"
    Erro crítico ou segurança
```

**Tab Examples**:
```markdown
=== "PHP 8.1+"
    ```php
    // Modern syntax
    ```

=== "PHP 8.0"
    ```php
    // Legacy syntax
    ```
```

**Code Block**:
```markdown
```php
// Always use PHP syntax highlighting
namespace App\Controller;

class ArticlesController extends AppController
{
    public function index()
    {
        // Code with Brazilian examples
        $this->set('artigos', $this->Articles->find('all'));
    }
}
` ``
```

## Key Differentiators
- 🇧🇷 Brazilian context (CPF validation, BR hosting, PT-BR date formats)
- 📊 Visual diagrams (Mermaid) for complex concepts
- 🔒 Security best practices emphasized
- 💡 Real-world examples (blog, e-commerce, API REST)
- 📱 Mobile-responsive design
- 🎨 CakePHP brand colors (#D33C44 red)

## Next Actions (if continuing documentation)
1. Expand docs/controllers-pagination.md (47 → 210+ lines)
2. Expand docs/controllers-components.md (74 → 340+ lines)
3. Expand docs/controllers-request-response.md (285 → 1000+ lines)
4. Start ORM section (docs/orm.md, docs/query-builder.md, docs/associations.md, etc.)
5. Add Development section (docs/application.md, docs/configuration.md, docs/routing.md)

## Git Info
- **Repository**: Not a git repo currently
- **Remote**: None
- **Branch**: N/A

---
**Last Updated**: 2025-11-16
**Context Size**: Optimized for LLM consumption (~1.5KB compressed knowledge)
