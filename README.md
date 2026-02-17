# AI-Augmented Databricks Workflows

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-brightgreen)](https://niveditanpatil.github.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A blog series exploring how AI coding assistants (Cursor, GitHub Copilot, etc.) can transform Databricks development workflows for data professionals.

**Live site:** [https://niveditanpatil.github.io](https://niveditanpatil.github.io)

## About This Blog

This blog covers practical workflows, setup guides, and best practices for using AI coding assistants with Databricks. Topics include:

- Development environment setup
- Remote cluster workflows with Databricks Connect
- MLflow experiment tracking
- Unity Catalog and data governance
- Prompt engineering for data engineering tasks
- CI/CD automation
- Team collaboration patterns

**Target Audience:** Data engineers, data scientists, ML engineers, and analytics engineers working with Databricks.

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
- **LinkedIn:** [your-profile](https://linkedin.com/in/yourprofile)
- **Blog:** [niveditanpatil.github.io](https://niveditanpatil.github.io)
