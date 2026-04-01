# Copilot Autofix Workflow & Metrics

> **Purpose:** This document describes the end-to-end Copilot Autofix workflow for reducing security vulnerabilities, and provides an inventory of the reporting surfaces available in the Enterprise Security Overview dashboard and via REST APIs.

---

## Autofix Workflow: Primary Path for Reducing Security Vulnerabilities with Copilot

**CodeQL → Copilot Autofix → Coding Agent → Resolved Vulnerability**

The workflow below describes how a security vulnerability flows from detection through automated fix suggestion to resolution.

### 1. Code Scanning

- The default scanner is **CodeQL**. Copilot Autofix is currently only available for CodeQL analysis.
- **Triggers:**
  | Trigger | When it runs |
  |---|---|
  | **PR creation / update** | On every pull request targeting a protected branch |
  | **Push to default branch** | On every push to `main` / `master`. Also catches historical vulnerabilities, dependency-based changes, or newly improved rules. |
  | **Scheduled runs** | Periodic scans that surface issues from updated rules, dependency changes, or previously undetected vulnerabilities |

> [!NOTE]
> **ESLint** is currently the only known third-party tool that works with Autofix, but that is outside the scope of security vulnerabilities. See [Copilot Autofix now supports partner code scanning tools](https://github.blog/changelog/) on the GitHub Changelog.

### 2. Alert Creation

- An **alert** is created for each vulnerability detected by CodeQL.

### 3. Autofix Suggestion Generation

This step only generates a **suggested fix** on the alert — it does not create a pull request. An Autofix suggestion can be generated in two ways:

1. **Manual** — A human clicks the **"Generate fix"** button on a supported alert.
2. **Security Campaign** — Autofix automatically generates suggestions for all applicable alerts in the campaign scope.

> The "Generate fix" button only appears if Autofix supports the given alert's rule.

<p align="center">
  <img src="images/image_3.png" alt="Code scanning alert showing the Generate fix button and Copilot Autofix for CodeQL" />
</p>

### 4. Coding Agent Assignment

- Once an Autofix suggestion exists, the **Copilot coding agent** can be assigned to implement the fix.
- The assignment option is **disabled** until a suggestion has been generated.

### 5. PR Review & Merge — Alert Resolved

- The coding agent (or developer) opens a pull request with the fix.
- CodeQL automatically rescans the PR.
- The standard code review and approval process is applied.
- The alert status transitions to **Fixed** after merge to the default branch.

---

## Reporting Inventory

The sections below catalog every reporting surface currently available for tracking Autofix and CodeQL metrics.

---

### Enterprise Security Overview Dashboard

> **Navigation:** *Enterprise → Security and quality → Overview*
>
> **Example URL:** <https://github.com/enterprises/octodemo/security/overview> | **Docs:** [Filtering alerts in security overview](https://docs.github.com/en/enterprise-cloud@latest/code-security/how-tos/manage-security-alerts/remediate-alerts-at-scale/filtering-alerts-in-security-overview)

---

#### Detection Tab

> **Example:** <https://github.com/enterprises/octodemo/security/overview?view=detection>

The **Detection** tab shows the current state of open alerts across the enterprise, including creation trends over time. Key metrics include open alerts over time grouped by severity (Critical, High, Medium, Low), age of alerts, reopened alerts, and secrets bypassed. It also provides an impact analysis with top repositories and vulnerabilities. Note that this tab does not contain any autofix-specific metrics.

<p align="center">
  <img src="images/AlertDetectionScreenshot.png" alt="Enterprise Security Overview — Detection tab showing open alerts over time, age of alerts, reopened alerts, and impact analysis" />
</p>

---

#### Remediation Tab

> **Example:** <https://github.com/enterprises/octodemo/security/overview?view=remediation>

The **Remediation** tab tracks how quickly and effectively **existing alerts on the default branch** are being closed across the enterprise, including closed alerts over time by severity, mean time to remediate, and net resolve rate. Unlike the Prevention tab (which tracks vulnerabilities caught in PRs before merge), Remediation focuses on vulnerabilities that have already been introduced. The Autofix-specific metric on this tab is:

| Metric | Description | Example Value |
|---|---|---|
| **Alerts fixed with autofix suggestions** | Count of alerts fixed with an accepted autofix out of all alerts with a suggested autofix | 14 of 307 |

<p align="center">
  <img src="images/image_1.png" alt="Enterprise Security Overview — Remediation tab showing closed alerts over time, mean time to remediate, net resolve rate, and alerts fixed with autofix suggestions" />
</p>

---

#### Prevention Tab

> **Example:** <https://github.com/enterprises/octodemo/security/overview?view=prevention>

The **Prevention** tab measures how many vulnerabilities are caught *before* reaching the default branch. It includes a Prevented vs. Introduced chart showing CodeQL vulnerabilities caught in the developer workflow, and the total CodeQL alerts fixed in pull requests. The Autofix-specific metrics on this tab are:

| Metric | Description | Example Value |
|---|---|---|
| **CodeQL PR alerts fixed with autofix suggestions** | Total CodeQL vulnerabilities fixed with an accepted autofix out of all with a suggested autofix | 1 of 6 |

<p align="center">
  <img src="images/image_2.png" alt="Enterprise Security Overview — Prevention tab showing prevented vs. introduced chart and CodeQL pull request autofix metrics" />
</p>

---

#### CSV Export

All Security Overview dashboards support **Export CSV**, which includes additional columns not shown in the UI. Notable columns in the export include:

| Column | Why it's useful |
|---|---|
| **PullRequestURL** | Direct link to the PR associated with each alert — not visible in the dashboard |
| **HasAutofix** | Whether an Autofix suggestion was generated for the alert |
| **AutofixAccepted** | Whether the Autofix suggestion was accepted |
| **ResolvedReason** | How the alert was resolved (e.g., fixed, dismissed) |
| **CodeQLRule** | The specific rule that triggered the alert |
| **Severity** | Alert severity level |
| **Repository / RepositoryID** | Identifies the repo — useful for cross-repo analysis |
| **CreatedAt / UpdatedAt** | Timestamps for tracking alert lifecycle |

This export is useful for building custom reports, filtering by autofix adoption, or correlating alerts with specific PRs.

---

### CodeQL Pull Request Alerts Page

> **Navigation:** *Enterprise → Security and quality → Insights → CodeQL pull requests*
>
> **Example URL:** <https://github.com/enterprises/octodemo/security/metrics/codeql>

This page provides a deep-dive into CodeQL alerts specifically within pull requests. At the top it shows summary metrics including total alerts found in PRs merged to protected branches, unique Autofix suggestions generated, and total alerts fixed. The Autofix-specific charts and breakdowns are:

#### Charts & Breakdowns

| Report | Details |
|---|---|
| **Alerts in pull requests** | Stacked chart broken down by: *Introduced and merged*, *Fixed with autofix*, *Fixed without autofix*, *Dismissed* |
| **Alerts fixed with autofix suggestions** | Count of alerts fixed with an accepted autofix out of all with a suggested autofix (e.g., 1 of 12) |
| **Remediation rates** | Side-by-side comparison of remediation percentages: with autofix vs. without autofix |
| **Mean time to remediate** | Side-by-side comparison of remediation time: with autofix vs. without autofix |
| **Most prevalent rules** | Top rules by frequency — e.g., *Unpinned tag for a non-immutable Action in workflow*, *Workflow does not contain permissions*, *Missing rate limiting*, *DOM text reinterpreted as HTML*, *Code injection* |

<p align="center">
  <img src="images/image_4.png" alt="CodeQL Pull Request Alerts dashboard showing alerts found, autofix suggestions, remediation rates, and mean time to remediate" />
</p>

---

### REST APIs

The following REST APIs can be used to query Autofix and pull request data programmatically:

#### Code Scanning API

The [Code Scanning REST API](https://docs.github.com/en/rest/code-scanning) includes endpoints for alerts, analyses, and autofixes. The key endpoints relevant to this workflow are:

**Alerts endpoints:**

| Endpoint | Relevance to this workflow |
|---|---|
| **[List code scanning alerts for a repo](https://docs.github.com/en/rest/code-scanning/code-scanning#list-code-scanning-alerts-for-a-repository)** | Query all alerts for a repository — filter by `state`, `tool`, or `severity`. The response includes `assignees` (check for `Copilot` to identify alerts assigned to the coding agent) and `fixed_at` to track resolution. |
| **[Get a code scanning alert](https://docs.github.com/en/rest/code-scanning/code-scanning#get-a-code-scanning-alert)** | Get full details for a single alert, including rule info, severity, location, and assignees. |

**Autofix endpoints:**

| Endpoint | Relevance to this workflow |
|---|---|
| **[Get the status of an autofix](https://docs.github.com/en/rest/code-scanning/code-scanning#get-the-status-of-an-autofix-for-a-code-scanning-alert)** | Most applicable for metrics — query whether an autofix suggestion exists for an alert and its current status |
| [Create an autofix](https://docs.github.com/en/rest/code-scanning/code-scanning#create-an-autofix-for-a-code-scanning-alert) | Programmatically triggers autofix generation for an alert (equivalent to clicking "Generate fix" in the UI) |
| [Commit an autofix](https://docs.github.com/en/rest/code-scanning/code-scanning#commit-an-autofix-for-a-code-scanning-alert) | Commits the generated autofix suggestion to a branch (creates the actual code change) |

> The **alerts** and **GET autofix** endpoints are the most relevant for reporting and metrics. The **Create** and **Commit** autofix endpoints are action-oriented — useful for automating the fix workflow itself rather than reporting on it.

#### Pull Requests API

The [Pull Requests REST API](https://docs.github.com/en/rest/pulls) is relevant because the coding agent creates PRs to implement autofix suggestions. You can use it to:

- **View PRs created by the coding agent** — Filter by author `Copilot` to find all PRs opened by the coding agent (`GET /repos/{owner}/{repo}/pulls?state=all`).
- **Link PRs back to alerts** — The coding agent includes a "Potential fix for alerts" section in the PR body with direct links to the associated code scanning alerts. You can retrieve this via the `body` field on `GET /repos/{owner}/{repo}/pulls/{pull_number}`.
- **Track PR resolution** — The PR response includes `state` (open/closed), `merged`, `merged_at`, and review status, letting you determine whether the fix has been reviewed, approved, and merged — which directly corresponds to Step 5 of the workflow (alert transitions to **Fixed** after merge).

> [!NOTE]
> There is no formal API field that directly links a PR to a code scanning alert. The connection exists in the PR body text as a convention set by the coding agent. To build a PR ↔ alert mapping programmatically, parse the PR body for `/security/code-scanning/{alert_number}` URLs.

<p align="center">
  <img src="images/PRBodyToParseScreenshot.png" alt="PR body showing 'Potential fix for alerts' section with a direct link to the code scanning alert" />
</p>

---

## Detailed Workflow Data by Phase — Funnel Analysis

This section frames the autofix workflow as a **conversion funnel**. At each phase, some alerts drop off — understanding where the biggest drop-offs occur reveals the bottleneck. For example:

- Few autofix suggestions generated → coverage or rule support gap
- Suggestions generated but not assigned → adoption problem
- PRs created but not merged → review bottleneck

The table below maps each phase to the available data sources for tracking this funnel. Where gaps in data exist, they are called out.

| Workflow Phase | What to measure | Data Source | Gap |
|---|---|---|---|
| **1. Code Scanning** | Scans running, languages analyzed | Code Scanning API — [List analyses](https://docs.github.com/en/rest/code-scanning/code-scanning#list-code-scanning-analyses-for-a-repository), [List CodeQL databases](https://docs.github.com/en/rest/code-scanning/code-scanning#list-codeql-databases-for-a-repository) | — |
| **2. Alert Creation** | Alerts created, severity, rule | Code Scanning API — [List alerts](https://docs.github.com/en/rest/code-scanning/code-scanning#list-code-scanning-alerts-for-a-repository); Dashboard — Remediation & Prevention tabs | — |
| **3. Autofix Suggestion** | Whether a suggestion was generated, when it started | Code Scanning API — [Get autofix status](https://docs.github.com/en/rest/code-scanning/code-scanning#get-the-status-of-an-autofix-for-a-code-scanning-alert) (`status`, `started_at`); Dashboard — "Alerts fixed with autofix suggestions" metric | No field on the alert itself indicating an autofix exists — requires a separate `/autofix` API call per alert |
| **4. Coding Agent Assignment** | Whether the coding agent was assigned to the alert | Code Scanning API — `assignees` field on [Get alert](https://docs.github.com/en/rest/code-scanning/code-scanning#get-a-code-scanning-alert) (look for `login: "Copilot"`) | No way to filter alerts by assignee in a single API call — must list alerts and check `assignees` client-side |
| **5. PR Created** | PR opened by coding agent, linked to alert | Pull Requests API — [List PRs](https://docs.github.com/en/rest/pulls/pulls#list-pull-requests) filtered by author `Copilot`; PR `body` contains alert links under a "Potential fix for alerts" section (e.g., `https://github.com/{owner}/{repo}/security/code-scanning/{alert_number}`) | No formal API field linking a PR to an alert — the alert URL is embedded in the PR `body` text and must be parsed |
| **5. PR Reviewed & Merged** | Review status, merge status, time to merge | Pull Requests API — `state`, `merged`, `merged_at`, reviews endpoint; Copilot Metrics API — `total_merged_created_by_copilot`, `median_minutes_to_merge_copilot_authored` | Requires first finding the PR (via CSV export `PullRequestURL`, or by searching PRs by author `Copilot` and parsing the body for the alert link) before you can query merge/review details |
| **5. Alert Resolved** | Alert transitioned to Fixed | Code Scanning API — `state: fixed`, `fixed_at` on the alert; Dashboard — Remediation tab (closed alerts, mean time to remediate) | The alert confirms resolution but does not link to the PR that fixed it — you only get `fixed_at`, not which PR triggered the fix |
| **End-to-end** | Full lifecycle from alert creation to resolution | CSV export from Dashboard — includes `PullRequestURL`, `HasAutofix`, `AutofixAccepted`, `ResolvedReason` in a single row per alert | CSV export provides the alert-to-PR link, but does not include PR merge status, review status, or time to merge — you still need to query the Pull Requests API using the `PullRequestURL` to get that data |

---

## High-Level Enterprise Autofix Metrics

The goal of this section is to understand at a high level how effective Autofix and Copilot are at reducing security vulnerabilities identified via CodeQL. The following metrics provide an enterprise-level view of autofix effectiveness. The table assesses whether each can be measured today.

| Autofix Metric | Description | Dashboard | API | Notes |
|---|---|---|---|---|
| **Alerts created (by date range)** | Total code scanning alerts created in a given period | **Yes** — [Remediation tab](https://github.com/enterprises/{enterprise}/security/overview?view=remediation) with date range selector (e.g., "Last 30 days" or custom) | **Yes** — Code Scanning API `List alerts` with `created_at` filtering | Dashboard supports restricting to current day or custom range |
| **Alerts resolved (by date range)** | Total alerts transitioned to Fixed in a given period | **Yes** — [Remediation tab](https://github.com/enterprises/{enterprise}/security/overview?view=remediation) "Closed alerts over time" chart with date range selector | **Yes** — Code Scanning API alerts with `state: fixed` and `fixed_at` date filtering | — |
| **Alerts fixed via autofix** | Alerts where an accepted autofix suggestion led to a fix | **Yes** — [Remediation tab](https://github.com/enterprises/{enterprise}/security/overview?view=remediation) "Alerts fixed with autofix suggestions" (e.g., 14 of 307) | **No** — No single API field; requires per-alert `/autofix` calls | Dashboard shows cumulative total within selected date range |
| **Alerts in pull requests breakdown** | How PR alerts were resolved: introduced and merged, fixed with autofix, fixed without autofix, dismissed | **Yes** — [CodeQL PR Alerts page](https://github.com/enterprises/{enterprise}/security/metrics/codeql) "Alerts in pull requests" stacked chart | **No** — No API provides this breakdown | Chart shows state-by-state breakdown over time within date range |
| **Autofix suggestions generated*** | Number of autofix suggestions created | **Yes** — [CodeQL PR Alerts page](https://github.com/enterprises/{enterprise}/security/metrics/codeql) "Unique Autofix suggestions" (e.g., 12) | **Partial** — API requires per-alert `/autofix` calls and checking `started_at` | Dashboard gives total count within date range; no daily granularity via API |
| **Autofix suggestions accepted** | Autofix suggestions that were accepted and applied | **Yes** — [CodeQL PR Alerts page](https://github.com/enterprises/{enterprise}/security/metrics/codeql) "Alerts fixed with autofix suggestions" (e.g., 1 of 12) | **No** — No API field for suggestion acceptance | Dashboard shows ratio within date range |
| **Remediation rate (with vs. without autofix)** | Comparison of fix rates for autofix vs. non-autofix alerts | **Yes** — [CodeQL PR Alerts page](https://github.com/enterprises/{enterprise}/security/metrics/codeql) "Remediation rates" side-by-side bar chart | **No** — No API provides this comparison | — |
| **Mean time to remediate (with vs. without autofix)** | Comparison of remediation speed for autofix vs. non-autofix alerts | **Yes** — [CodeQL PR Alerts page](https://github.com/enterprises/{enterprise}/security/metrics/codeql) "Mean time to remediate" side-by-side | **No** — No API provides this comparison | — |
| **Prevented vs. Introduced** | Vulnerabilities caught in PRs before reaching the default branch | **Yes** — [Prevention tab](https://github.com/enterprises/{enterprise}/security/overview?view=prevention) chart with date range selector | **No** — No API equivalent | — |
| **Most prevalent rules** | Top CodeQL rules by frequency | **Yes** — [CodeQL PR Alerts page](https://github.com/enterprises/{enterprise}/security/metrics/codeql) "Most prevalent rules" list | **No** — No API provides rule-level aggregation | — |

> **\*** **Autofix suggestions generated** — This metric may appear low if the organization is not using [security campaigns](https://docs.github.com/en/code-security/securing-your-organization/fixing-security-alerts-at-scale/about-security-campaigns). Without a campaign (which automatically generates fixes for targeted alerts in bulk), autofix suggestion generation is gated on an individual developer manually clicking the **"Generate fix"** button on each alert. Low suggestion counts therefore may reflect an adoption gap rather than a coverage or capability gap.

---

## Open Decision Points

The following questions need to be settled before finalizing the metrics strategy:

1. **Is this the right workflow?** Is the workflow identified in this document (CodeQL → Copilot Autofix → Coding Agent → PR → Resolved Vulnerability) what most customers want to monitor and measure to understand reduction of security vulnerabilities using Copilot? Are there alternative or additional workflows we should account for?

2. **Are these the right metrics?** Are the high-level enterprise metrics and detailed workflow phase metrics listed in this document what customers need to understand Autofix/Copilot effectiveness? Are there metrics missing that customers would expect, or metrics listed here that are not actually useful?