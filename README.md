# CEF GitHub Action

A composite GitHub Action that forwards pull request events to a [CEF (Cere Edge Framework)](https://cef.ai) agent service for processing via `@cere-ddc-sdk/client`. The CEF agent handles all enrichment server-side (files changed, commits, diffs, etc.) and can write execution logs to Notion.

## What It Does

On every PR open, close, merge, or update, this action:

1. Reads the GitHub PR event payload from the runner
2. Packages it into a CEF event and sends it to your agent via `@cere-ddc-sdk/client`
3. The CEF agent handles enrichment, processing, and optional Notion logging

## Usage

Add a workflow file to any repo you want to track:

```yaml
# .github/workflows/cef-pr-tracker.yml
name: CEF PR Tracker

on:
  pull_request:
    types: [opened, closed, reopened, synchronize]

jobs:
  send-pr-event:
    runs-on: ubuntu-latest
    steps:
      - uses: cere-io/cef-github-action@v1
        with:
          wallet_uri:     ${{ secrets.CEF_WALLET_URI }}
          ddc_base_url:   ${{ secrets.CEF_DDC_BASE_URL }}
          agent_service:  ${{ secrets.CEF_AGENT_SERVICE }}
          workspace:      ${{ secrets.CEF_WORKSPACE }}
          # Optional:
          notion_page_id: ${{ secrets.NOTION_PAGE_ID }}
          notion_api_key: ${{ secrets.NOTION_API_KEY }}
          event_type:     "GITHUB_ACTION_PR_EVENT"
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `wallet_uri` | Yes | — | CEF wallet URI (Ed25519 signer) — identifies the sender |
| `ddc_base_url` | Yes | — | DDC Base URL |
| `agent_service` | Yes | — | Agent service public key |
| `workspace` | Yes | — | CEF Workspace ID |
| `stream` | No | `""` | CEF Stream ID |
| `event_type` | No | `GITHUB_ACTION_PR_EVENT` | CEF event type identifier |
| `github_token` | No | `github.token` | GitHub token forwarded to the CEF agent for server-side API calls |
| `notion_page_id` | No | `""` | Notion page ID where the execution log is written |
| `notion_api_key` | No | `""` | Notion integration API key |

## Secrets Setup

In the consuming repo, go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|---|---|
| `CEF_WALLET_URI` | Ed25519 wallet URI |
| `CEF_DDC_BASE_URL` | DDC Base URL |
| `CEF_AGENT_SERVICE` | Agent service public key |
| `CEF_WORKSPACE` | Workspace ID |
| `NOTION_PAGE_ID` | *(optional)* Notion page for log output |
| `NOTION_API_KEY` | *(optional)* Notion integration key |

Contact the CEF platform team to get your `wallet_uri`, `ddc_base_url`, `agent_service`, and `workspace` values.

## Payload Sent to CEF

```json
{
  "event_type": "GITHUB_ACTION_PR_EVENT",
  "action": "synchronize",
  "merged": false,
  "pr_number": 42,
  "pr_title": "feat: add new feature",
  "pr_url": "https://github.com/org/repo/pull/42",
  "pr_body": "...",
  "repo": "org/repo",
  "author": "username",
  "merged_by": null,
  "merged_at": null,
  "base_branch": "main",
  "before": "abc1234...",
  "after": "def5678...",
  "timestamp": "2026-03-05T10:00:01Z",
  "delivery_id": "org/repo#42#1741168801000",
  "github_token": "***",
  "notion_page_id": "abc123",
  "notion_api_key": "***"
}
```

The `github_token` is forwarded so the CEF agent can call the GitHub API server-side for enrichment (diff, file list, commits).

## Requirements

- Node.js 20 (automatically set up by the action)
- A running CEF agent service configured to handle the event type
