---
title: "Setting Up Databricks Extension in VS Code and Cursor: A Step-by-Step Guide"
date: 2026-02-18 10:00:00 -0500
categories: [setup, databricks]
tags: [databricks, cursor, vscode, ide, setup, development-environment]
excerpt: "Learn how to set up the Databricks extension in VS Code, Cursor, or any VS Code-based IDE for local development with remote clusters. Includes OAuth setup, Python environment configuration, and troubleshooting tips."
---

I recently set up the Databricks extension in Cursor to work on data pipelines locally while executing against remote clusters. Here's what I learned—including the gotchas that took me way too long to figure out.

> 📝 **Note**: This guide works for **VS Code, Cursor, and any VS Code-based IDE** (like VSCodium). The Databricks extension is compatible with all of them since they share the same extension API.

## Why Use the Extension?

The Databricks extension lets you:
- Write code in Cursor with full IDE features (autocomplete, linting, etc.)
- Execute code on remote Databricks clusters via Databricks Connect
- Debug notebooks cell-by-cell locally
- Avoid constant context switching to the Databricks web UI

Source: [Databricks Extension Documentation](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/)

## Prerequisites

| Requirement | Version | Why You Need It |
|-------------|---------|-----------------|
| VS Code-compatible IDE | Latest | VS Code, Cursor, VSCodium, etc. |
| Python | 3.12+ | Required for Databricks Connect |
| Databricks Workspace | Active | Need cluster access |
| pip | Latest | For installing packages in venv |

---

## Step 1: Install the Extension

1. Open your IDE (VS Code, Cursor, etc.) and go to Extensions (⌘+Shift+X on Mac, Ctrl+Shift+X on Windows/Linux)
2. Search for **"Databricks"** (publisher: `databricks`)
3. Click Install

The extension enables connection to remote workspaces and execution on clusters or serverless compute.[^1] It works identically across all VS Code-based IDEs.

---

## Step 2: Authentication

The extension uses [Databricks unified authentication](https://docs.databricks.com/aws/en/dev-tools/auth/unified-auth) with automatic token refresh.

### Option A: OAuth (Recommended)

> ✅ **Recommended** - Databricks recommends OAuth interactive authorization, which is easy to configure.[^2]

**Steps:**
1. Click Databricks icon in sidebar
2. Configuration view → **Auth Type** → gear icon
3. Select **OAuth (user to machine)**
4. Enter profile name → **Login to Databricks**
5. Complete browser authentication
6. Allow **all-apis** access if prompted

### Option B: Personal Access Token

For automated/CI scenarios:

1. Configuration → **Auth Type** → gear icon  
2. Select **Personal Access Token**
3. Generate PAT in Databricks workspace → paste token

> 📝 **Note**: Extension auto-creates `.databricks/` folder with `databricks.env` and adds it to `.gitignore`.[^2]

### Option C: Existing Profile

If you have `~/.databrickscfg`:
1. **Auth Type** → gear icon  
2. Select your existing profile

---

## Step 3: Python Environment (Critical!)

The extension needs Python 3.12+ with pip for Databricks Connect.[^1]

### Using uv (Faster)

[uv](https://github.com/astral-sh/uv) is faster and cleaner for creating venvs:

```bash
cd /path/to/your/workspace

# Create venv with Python 3.12
uv venv --python 3.12 .venv

# ⚠️ CRITICAL: Install pip (uv doesn't include it by default)
uv pip install pip --python .venv/bin/python

# Verify
.venv/bin/python --version  # Should show 3.12.x
.venv/bin/python -m pip --version
```

> ⚠️ **Common Gotcha**: I spent 30 minutes debugging "No module named pip" because `uv` doesn't include pip by default. Don't skip that second command!

### Using Standard venv

```bash
cd /path/to/your/workspace

python3.12 -m venv .venv

# Verify
.venv/bin/python --version
.venv/bin/python -m pip --version
```

### Configure Cursor

Add to `.vscode/settings.json`:

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python"
}
```

Reload Cursor after this.

---

## Step 4: Create/Open Databricks Project

Three options:

| Method | When to Use | Command |
|--------|-------------|---------|
| **Create New** | Starting from scratch | Databricks sidebar → "Create a new project" |
| **Open Existing** | Has `databricks.yml` | File → Open Folder |
| **Convert Existing** | Add Databricks to existing project | Databricks sidebar → "Create configuration" |

### Manual Configuration (Optional)

Create `databricks.yml` in project root:

```yaml
bundle:
  name: your_project_name

workspace:
  host: https://your-workspace.cloud.databricks.com

targets:
  dev:
    workspace:
      root_path: /Workspace/Users/your.email@company.com/your/path
    default: true
```

Source: [Databricks Extension Authentication](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/authentication)

> 💡 **Tip**: If prompted about Python environment, click "Use Existing" and select your `.venv`.

---

## Step 5: Select Cluster

In the Databricks sidebar under **CONFIGURATION**:

1. Verify **Auth Type** shows your profile
2. Click **Cluster** → select target cluster from dropdown

---

## Step 6: Cluster Tagging (Important for Teams!)

> 💡 **Real-world tip**: A data engineer on my team asked about this—when running jobs via the extension, how do we trace usage back to individual users for billing and debugging?

The extension doesn't have UI controls for cluster tags, but there are ways to ensure your clusters have proper tags like `owner`, `project`, `sub-project`:

### Why Cluster Tags Matter

- **Billing attribution**: Track costs by team/project
- **Debugging**: Trace unexpected compute spikes to specific users
- **Governance**: Enforce resource allocation policies
- **Compliance**: Some orgs require tagging for audit trails

### How to Tag Clusters

| Approach | How It Works |
|----------|-------------|
| **Cluster Policy** | If your workspace has a policy that sets/requires custom tags, any cluster created under that policy (including from the extension) gets those tags automatically. No action needed on your part. |
| **Use Pre-Tagged Cluster** | Create cluster via Databricks UI or Terraform with `custom_tags`, then select it in the extension. Tags are already set. |
| **Bundle-Defined Cluster** | Define cluster in `databricks.yml` with tags, deploy bundle, then select in extension. |

### Option 3 Example: Bundle-Defined Cluster

Add to your `databricks.yml`:

```yaml
resources:
  clusters:
    dev-cluster:
      name: my-project-dev
      spark_version: "14.3.x-scala2.12"
      node_type_id: i3.xlarge
      autotermination_minutes: 20
      custom_tags:
        owner: "your.email@company.com"
        project: "YourProject"
        sub-project: "SubProject"
```

Deploy with `databricks bundle deploy`, then select **my-project-dev** in the extension's Cluster dropdown.

> ⚠️ **Don't use tag key `Name`** - Databricks sets this automatically. See [Use tags to attribute and track usage](https://docs.databricks.com/aws/en/admin/account-settings/usage-detail-tags).

**For Terraform users**: Use `custom_tags` on cluster resources. For jobs, use `tags = local.required_tags` on the job resource, and add `custom_tags` in the `new_cluster` block if needed.

---

## Step 7: Run Your Code

Right-click your `.py` file → **"Run on Databricks"**:

| Option | Description | Best For |
|--------|-------------|----------|
| **Run with Databricks Connect** | Runs locally, executes on remote cluster | Interactive development, immediate output |
| **Debug with Databricks Connect** | Same + debugging enabled | Troubleshooting with breakpoints |
| **Upload and Run File** | Uploads to workspace, runs there | Production-like testing |
| **Run File as Workflow** | Submits as Databricks job | Background/scheduled tasks |

**My recommendation**: Use **Databricks Connect** for development—output streams directly to your terminal.

Source: [Databricks Extension Documentation](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/)

---

## Bonus: Keep Laptop Awake for Long Jobs

Databricks Connect requires an active connection. For long-running jobs, prevent sleep:

```bash
# Find your Databricks Connect process
ps aux | grep dbconnect-bootstrap | grep -v grep

# Keep laptop awake until process completes (replace <PID>)
caffeinate -i -w <PID>
```

The `-i` flag prevents idle sleep, `-w <PID>` exits automatically when done.

> 💡 **For overnight jobs**: Use "Run File as Workflow" instead—runs entirely on Databricks servers.

---

## Troubleshooting

### "Failed to set up Python environment"

**Check Python version and pip:**

```bash
.venv/bin/python --version  # Must be 3.12+
.venv/bin/python -m pip --version
```

If pip is missing (common with `uv`):

```bash
uv pip install pip --python .venv/bin/python
```

### "Databricks Connect disabled" in Status Bar

Click it → select your Python 3.12 venv when prompted.

### "A Databricks project was not detected"

Create `databricks.yml` in project root. Extension requires this file.

### SSL/Certificate Errors

Corporate networks may intercept SSL:
- Use extension UI instead of CLI
- Upload notebooks manually via web interface

### "[TABLE_OR_VIEW_NOT_FOUND]"

Use full 3-level Unity Catalog format: `catalog.schema.table`

---

## File Structure Example

After setup, your workspace should look like:

```
workspace/
├── .venv/                    # Python 3.12 venv
├── .vscode/
│   └── settings.json         # Python interpreter path
├── project1/
│   ├── .databricks/          # Auto-created, gitignored
│   │   └── databricks.env
│   ├── databricks.yml        # Project config
│   └── notebook.py           # Your code
└── project2/
    ├── .databricks/
    ├── databricks.yml
    └── pipeline.py
```

---

## Key Takeaways

1. **Python 3.12+ is mandatory** - Extension won't work without it
2. **If using `uv`, manually install pip** - Would've saved me a debugging headache
3. **OAuth is simpler than PAT** - Automatic token refresh
4. **Use Databricks Connect for development** - Immediate feedback in terminal
5. **Keep `.databricks/` gitignored** - Extension handles this automatically

---

## Next Steps

Now that the extension is set up, I'm planning to:
- Measure productivity impact (baseline vs. with extension)
- Build custom Cursor rules for Databricks patterns
- Set up CI/CD with the extension

I'll be documenting these experiments in future posts, so stay tuned!

---

## Get In Touch

Have you tried the Databricks extension in VS Code or Cursor? Run into different issues or have tips to share? I'd love to hear about your experience:

- **LinkedIn**: [linkedin.com/in/patilnivedita](https://linkedin.com/in/patilnivedita)
- **GitHub**: [github.com/niveditanpatil](https://github.com/niveditanpatil)

---

## References

[^1]: [Databricks Extension Documentation](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/)
[^2]: [Set up OAuth authorization](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/authentication)
