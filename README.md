# Learning in Public: Data & AI

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-brightgreen)](https://niveditanpatil.github.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Documenting experiments with AI coding tools, data platforms, and whatever else I'm learning about data engineering, data science, machine learning, and analytics.

**Live site:** [https://niveditanpatil.github.io](https://niveditanpatil.github.io)

## About This Blog

A learning journal covering experiments, workflows, and lessons learned while working with modern data tools and AI coding assistants. Topics include:

- AI coding assistants (Cursor, GitHub Copilot, etc.)
- Data platform workflows and setup
- MLflow and experiment tracking
- Data engineering and analytics patterns
- Prompt engineering for data tasks
- What works (and what doesn't)

**Audience:** Data scientists, data engineers, ML engineers, analytics engineers, and anyone curious about modern data tools.

## Repository Structure

```
.
├── _posts/              # Blog posts (Markdown files)
├── _config.yml          # Jekyll configuration
├── assets/
│   ├── images/         # Blog images
│   └── css/            # Custom CSS (if needed)
├── about.md            # About page
├── index.md            # Home page
├── Gemfile             # Ruby dependencies
└── README.md           # This file
```

## Local Development

### Prerequisites

- Ruby 2.7+ installed
- Bundler gem installed

### Setup

```bash
# Clone the repository
git clone https://github.com/niveditanpatil/niveditanpatil.github.io.git
cd niveditanpatil.github.io

# Install dependencies
bundle install

# Serve locally
bundle exec jekyll serve

# View at http://localhost:4000
```

### Creating a New Post

Create a new file in `_posts/` with the format: `YYYY-MM-DD-title.md`

```markdown
---
title: "Your Post Title"
date: 2026-02-17 10:00:00 -0500
categories: [category1, category2]
tags: [tag1, tag2, tag3]
excerpt: "Brief description for SEO and previews"
---

Your content here...
```

## Companion Resources

- **Code Repository:** [databricks-ai-workflows](https://github.com/niveditanpatil/databricks-ai-workflows) - Code samples, Cursor rules, CI/CD templates

## Built With

- [Jekyll](https://jekyllrb.com/) - Static site generator
- [Minimal Mistakes Theme](https://mmistakes.github.io/minimal-mistakes/) - Jekyll theme
- [GitHub Pages](https://pages.github.com/) - Hosting

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contact

- **GitHub:** [@niveditanpatil](https://github.com/niveditanpatil)
- **LinkedIn:** [patilnivedita](https://linkedin.com/in/patilnivedita)
- **Blog:** [niveditanpatil.github.io](https://niveditanpatil.github.io)
