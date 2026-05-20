# starter

A template repository for dispatching AI agent tasks via GitHub Issues.

Use this template to spin up a new repo that can send tasks to the `daulton-agent` Cloudflare Worker — models read your codebase, write reviews, open PRs, or push fixes directly.

---

## How it works

1. Create a new repo from this template.
2. Open an issue using the **Agent Task** template.
3. Fill in the model, mode, and task description.
4. The `daulton-agent` webhook picks up the issue and either posts a review inline or dispatches a job to `agent-runner`.

---

## Customization guide

### Adding or changing models

Models are defined in two places. You need to keep them in sync:

#### 1. Issue template — `daultonjenk/daulton-personal-website`

File: `.github/ISSUE_TEMPLATE/agent-task.yml`

The `options` list under the `Agent` dropdown is what users see when opening an issue. Add a new entry here:

```yaml
options:
  - Gemini 3.1 Flash Lite          # existing
  - My New Model Label              # add your entry here
```

The label is just a human-readable display name — it can be anything.

#### 2. Worker model map — `daultonjenk/daulton-personal-website`

File: `workers/daulton-agent/src/index.js`

Find the `parseIssueForm` function and the `modelMap` object inside it. Add a matching entry:

```javascript
const modelMap = {
  // ... existing entries ...
  'My New Model Label': 'provider/model-id-on-openrouter',
};
```

The key **must exactly match** the string in the issue template dropdown. The value is the OpenRouter model ID — find these at [openrouter.ai/models](https://openrouter.ai/models), click a model, and copy its API name (shown as `provider/model-name`).

After editing both files, commit and push to `main` — the deploy workflow will publish the updated worker automatically.

---

### Adding labels

The `agent-task` label is created automatically by `.github/workflows/setup-labels.yml` when the repo is first pushed to. For any other custom labels:

1. Go to **Issues → Labels → New label** in the GitHub UI, or
2. Add another label creation step to `setup-labels.yml`:

```yaml
- name: Create my-label label
  run: |
    gh label create my-label --color "AABBCC" --description "My label" --force
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Color values are 6-digit hex codes without the `#`.

---

## Manual setup steps (first time only)

After creating a repo from this template:

1. Make sure the `daulton-agent` GitHub App is **installed on the new repo** — go to GitHub → your account → Settings → Applications → daulton-agent → Repository access, and add the repo.
2. That's it — the webhook fires automatically for any new issue with the `agent-task` label.

---

## Model reference

Current model options and their OpenRouter IDs:

| Display name | OpenRouter ID |
|---|---|
| Gemini 3.1 Flash Lite | `google/gemini-3.1-flash-lite` |
| Gemini 3.1 Flash Lite (Preview) | `google/gemini-3.1-flash-lite-preview` |
| Gemini Flash (latest) | `~google/gemini-flash-latest` |
| Gemini Pro (latest) | `~google/gemini-pro-latest` |
| Kimi K2.6 | `moonshotai/kimi-k2.6` |
| Qwen3.6 Flash | `qwen/qwen3.6-flash` |
| Qwen3.6 27B | `qwen/qwen3.6-27b` |
| Qwen3.6 35B A3B | `qwen/qwen3.6-35b-a3b` |
| Qwen3.6 Plus | `qwen/qwen3.6-plus` |
| Qwen3.6 Max Preview | `qwen/qwen3.6-max-preview` |
| DeepSeek V4 Flash | `deepseek/deepseek-v4-flash` |
| DeepSeek V4 Pro | `deepseek/deepseek-v4-pro` |

> **Note on Gemini "latest" aliases**: The `~` prefix is how OpenRouter exposes tracked aliases that always point to the newest stable version. These are not pinned version numbers, so the underlying model may change over time.
