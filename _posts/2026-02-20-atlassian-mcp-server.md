---
title: "Setting Up Atlassian MCP Server: Jira and Confluence Access from Your IDE"
date: 2026-02-20 10:00:00 -0500
categories: [setup, atlassian, ai-tools]
tags: [atlassian, jira, confluence, cursor, mcp, model-context-protocol, productivity]
excerpt: "Learn how to install the Atlassian MCP server to give your AI coding assistant direct access to Jira issues and Confluence pages - create tickets, search issues, and read documentation without leaving your IDE."
---

After setting up the [Databricks AI Dev Kit MCP server](/2026/02/19/databricks-ai-dev-kit-mcp/), I wanted to streamline my Jira workflow. The [Atlassian MCP server](https://mcp.atlassian.com/) lets your AI assistant interact with Jira and Confluence directly from your IDE.

Here's what I learned setting it up, including a tricky configuration issue I ran into.

## Table of Contents
- [TL;DR: What I Liked and Didn't Like](#tldr-what-i-liked-and-didnt-like)
- [What Is the Atlassian MCP Server?](#what-is-the-atlassian-mcp-server)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [What You Get](#what-you-get)
- [Verification](#verification)
- [Troubleshooting: The Hidden Config File Issue](#troubleshooting-the-hidden-config-file-issue)
- [Real-World Usage](#real-world-usage)
- [Next Steps](#next-steps)

## TL;DR: What I Liked and Didn't Like

| ✅ What I Liked | ❌ What I Didn't Like |
|----------------|---------------------|
| **No local installation needed** - Runs as a remote service via `mcp-remote`, no Python/Node dependencies to manage | **Configuration conflict was cryptic** - Had to dig through Cursor internals to discover project-level vs global config precedence |
| **28 tools for Jira and Confluence** - Create issues, search with JQL, read Confluence pages, add comments, transition tickets, etc. | **Requires full Cursor restart** - Reloading the window isn't enough; you must Cmd+Q and reopen |
| **OAuth authentication built-in** - Just sign in through Atlassian, no API tokens to manage | **Project-level config naming is verbose** - Server becomes `project-0-GitRepos-atlassian` instead of just `atlassian` |
| **Works alongside other MCP servers** - Can use Databricks and Atlassian MCPs simultaneously | **Limited error messages** - When it's not working, the errors don't tell you *why* |

## What Is the Atlassian MCP Server?

The Atlassian MCP server is a remote MCP service hosted by Atlassian that gives your AI coding assistant access to:

- **Jira**: Create issues, search with JQL, transition tickets, add comments, manage worklogs
- **Confluence**: Read pages, search with CQL, create/update pages, add comments

Unlike the Databricks MCP server (which runs locally), the Atlassian server runs remotely and connects via Server-Sent Events (SSE).

## Prerequisites

You'll need:

- **Cursor** (or another IDE with MCP support)
- **npx** (comes with Node.js) - Used to run `mcp-remote`
- **Atlassian Cloud account** with access to Jira and/or Confluence
- **Permissions** to access the Jira projects and Confluence spaces you want to query

Check if you have npx:

```bash
npx --version
# Should show a version number like 10.x.x
```

If not, [install Node.js](https://nodejs.org/) which includes npx.

## Installation

### Step 1: Configure the MCP Server

If you already have other MCP servers configured (like Databricks), you'll need to **merge** the configurations. This is important!

Check if you have an existing project-level config:

```bash
ls ~/.cursor/mcp.json              # Global config
ls /path/to/your/project/.cursor/mcp.json   # Project-level config
```

**If you have a project-level `.cursor/mcp.json` file**, edit that one (project configs take precedence).

**If you only have a global config** or no config at all, you can start with either location.

Add the Atlassian server to your `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/sse"]
    }
  }
}
```

**If you already have other servers** (like Databricks), merge them:

```json
{
  "mcpServers": {
    "databricks": {
      "command": "/Users/yourusername/.ai-dev-kit/.venv/bin/python",
      "args": ["/Users/yourusername/.ai-dev-kit/repo/databricks-mcp-server/run_server.py"],
      "env": {
        "DATABRICKS_CONFIG_PROFILE": "cursor"
      }
    },
    "atlassian": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/sse"]
    }
  }
}
```

### Step 2: Restart Cursor Completely

**Important:** You must fully quit and restart Cursor (Cmd+Q on Mac, not just reload window).

1. Quit Cursor completely (Cmd+Q or File → Quit)
2. Reopen Cursor
3. Open a file from your workspace to activate it

### Step 3: Enable the Server in Settings

1. Open **Cursor Settings** (Cmd+, or Ctrl+,)
2. Search for **MCP**
3. Find the **Atlassian** server entry
4. Toggle it **ON**

You should see it listed with 28 tools.

### Step 4: Authenticate with Atlassian

The first time your AI assistant tries to use the Atlassian MCP server, you'll be prompted to:

1. Sign in to your Atlassian account
2. Grant access to your Jira and Confluence sites
3. Authorize the MCP server

Follow the OAuth flow in your browser to complete authentication.

## What You Get

Once installed, your AI assistant has access to 28 MCP tools:

### Jira Tools (18 tools)

| Tool | What It Does |
|------|--------------|
| `getJiraIssue` | Get details for a specific issue (e.g., MOC-123) |
| `createJiraIssue` | Create a new issue (Task, Bug, Story, etc.) |
| `editJiraIssue` | Update issue fields (assignee, status, description, etc.) |
| `searchJiraIssuesUsingJql` | Search issues with JQL (Jira Query Language) |
| `transitionJiraIssue` | Move issue to different status (e.g., In Progress → Done) |
| `addCommentToJiraIssue` | Add a comment to an issue |
| `addWorklogToJiraIssue` | Log time spent on an issue |
| `getTransitionsForJiraIssue` | Get available status transitions for an issue |
| `getJiraIssueRemoteIssueLinks` | Get remote links associated with an issue |
| `getVisibleJiraProjects` | List projects you have access to |
| `getJiraProjectIssueTypesMetadata` | Get issue types for a project |
| `getJiraIssueTypeMetaWithFields` | Get field metadata for an issue type |
| `lookupJiraAccountId` | Find Atlassian account ID by email |

### Confluence Tools (10 tools)

| Tool | What It Does |
|------|--------------|
| `getConfluencePage` | Get page content by ID (markdown or ADF format) |
| `getConfluenceSpaces` | List spaces you have access to |
| `getPagesInConfluenceSpace` | List pages in a specific space |
| `getConfluencePageDescendants` | Get child pages of a parent page |
| `searchConfluenceUsingCql` | Search Confluence with CQL (Confluence Query Language) |
| `createConfluencePage` | Create a new page |
| `updateConfluencePage` | Update existing page content |
| `createConfluenceInlineComment` | Add inline comment to page content |
| `createConfluenceFooterComment` | Add footer comment to page |
| `getConfluencePageInlineComments` | Get inline comments on a page |
| `getConfluencePageFooterComments` | Get footer comments on a page |

### General Tools

- `getAccessibleAtlassianResources` - List accessible Jira/Confluence sites
- `atlassianUserInfo` - Get your Atlassian user information
- `search` - General search across Atlassian resources
- `fetch` - Generic HTTP fetch for Atlassian APIs

## Verification

To verify the MCP server is working, ask your AI assistant to list your Jira projects:

**Example prompt:**
> "List all Jira projects I have access to"

If it's working, your assistant will call `getVisibleJiraProjects` and return your projects.

You can also test issue searches:

> "Search for all open issues assigned to me in Jira"

The assistant should use `searchJiraIssuesUsingJql` with JQL: `assignee = currentUser() AND status != Done`

## Troubleshooting: The Hidden Config File Issue

### Problem

After adding the Atlassian MCP server, Cursor settings showed it as enabled with 28 tools, but when I asked my AI assistant to use it, I got:

```
Error: MCP server does not exist: atlassian
Available servers: cursor-ide-browser, project-0-GitRepos-databricks
```

### Root Cause

I had **two separate `mcp.json` files**:

- **Global config**: `~/.cursor/mcp.json` with Atlassian config
- **Project config**: `/Users/<your-username>/GitRepos/.cursor/mcp.json` with Databricks config

**Cursor uses project-level config when a workspace is open**, ignoring the global config. So the AI assistant couldn't see the Atlassian server.

### Solution

Merge both configs into the **project-level** `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "databricks": {
      "command": "/Users/<your-username>/.ai-dev-kit/.venv/bin/python",
      "args": ["/Users/<your-username>/.ai-dev-kit/repo/databricks-mcp-server/run_server.py"],
      "env": {
        "DATABRICKS_CONFIG_PROFILE": "cursor"
      }
    },
    "atlassian": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/sse"]
    }
  }
}
```

Then:
1. **Fully quit Cursor** (Cmd+Q)
2. **Reopen Cursor**
3. **Open a workspace file** to activate the project

Now the AI assistant can access both servers.

### Key Lessons

- **Project-level config takes precedence** - If you have `/path/to/project/.cursor/mcp.json`, that's what gets used
- **Global config is ignored** when a project config exists
- **Cursor's UI can be misleading** - Settings may show servers from global config even if they're not accessible
- **Server naming changes** - Project MCP servers get prefixed: `atlassian` becomes `project-0-GitRepos-atlassian`
- **Full restart required** - Reloading the window isn't enough

## Real-World Usage

Here are some things I've used the Atlassian MCP server for:

### Creating Jira Issues

**Me:** "Create a Jira task in the MOC project: 'Update data pipeline documentation' with description 'Add examples for new transformations'"

**Cursor (via MCP):**

Calls: `createJiraIssue` with:
- `projectKey`: "MOC"
- `issueTypeName`: "Task"
- `summary`: "Update data pipeline documentation"
- `description`: "Add examples for new transformations"

Returns: `MOC-123` created successfully

### Searching Issues with JQL

**Me:** "Show me all high-priority bugs in the DATA project that are unassigned"

**Cursor (via MCP):**

Calls: `searchJiraIssuesUsingJql` with:
```
jql: "project = DATA AND issuetype = Bug AND priority = High AND assignee is EMPTY"
```

Returns list of matching issues with summaries, statuses, and creation dates.

### Reading Confluence Documentation

**Me:** "Get the content of Confluence page 123456789 in markdown format"

**Cursor (via MCP):**

Calls: `getConfluencePage` with:
- `pageId`: "123456789"
- `contentFormat`: "markdown"

Returns the page content, which the assistant can then summarize or reference.

### Transitioning Issues

**Me:** "Move MOC-456 to In Progress"

**Cursor (via MCP):**

1. Calls `getTransitionsForJiraIssue` to find available transitions
2. Calls `transitionJiraIssue` with the "In Progress" transition ID

Returns confirmation of status change.

## Next Steps

Now that you have the Atlassian MCP server set up:

1. **Try creating issues** - Ask your assistant to create tickets for bugs or tasks
2. **Search with JQL** - Use complex Jira queries without opening the browser
3. **Read Confluence docs** - Pull in documentation context directly into your coding workflow
4. **Automate worklogs** - Log time on issues programmatically
5. **Integrate with other MCP servers** - Combine with Databricks MCP for end-to-end workflows

The Atlassian MCP server is particularly useful when you're:
- Writing code and want to reference Jira tickets without context-switching
- Need to create issues based on code analysis or errors
- Want to pull in Confluence documentation as context for your AI assistant
- Tracking time or updating issue status as you complete work

---

## Want to Connect?

If you're using MCP servers to streamline your workflow, I'd love to hear what you're building with them. Connect with me on [LinkedIn](https://linkedin.com/in/patilnivedita) or check out my code on [GitHub](https://github.com/niveditanpatil).

---

## References

- [Atlassian MCP Server](https://mcp.atlassian.com/)
- [Model Context Protocol (MCP) Specification](https://modelcontextprotocol.io/)
- [Previous post: Databricks AI Dev Kit MCP Server](/2026/02/19/databricks-ai-dev-kit-mcp/)
- [Jira Query Language (JQL) Documentation](https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/)
- [Confluence Query Language (CQL) Documentation](https://developer.atlassian.com/server/confluence/advanced-searching-using-cql/)
