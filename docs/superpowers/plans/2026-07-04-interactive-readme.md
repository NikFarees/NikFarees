# Interactive Profile README Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rework `NikFarees/NikFarees` profile README to feel interactive (typing banner, collapsible stack, live stats, contribution snake, WakaTime) without client-side JS, and replace `---` dividers with a gradient SVG rule.

**Architecture:** Static `README.md` edits + a local `assets/divider.svg` asset + two GitHub Actions workflows (`snake.yml`, `waka-readme.yml`) that regenerate live content on a schedule. All "dynamic" content is either a third-party badge `<img>` (no secret) or repo-hosted SVG/README block kept fresh by Actions.

**Tech Stack:** GitHub-flavored Markdown + raw HTML (`<details>`, `<picture>`, `<table>`), SVG, GitHub Actions (YAML), `Platane/snk@v3`, `anmol098/waka-readme-stats@master`, `github-readme-stats` / `github-readme-streak-stats` public badge APIs.

## Global Constraints

- Repo is `NikFarees/NikFarees` (confirmed via `git remote -v`) — all repo-relative URLs (raw.githubusercontent.com, workflow branch refs) use this owner/repo.
- No `<script>` tags, no `<style>` blocks — GitHub strips them on render.
- No plain `---` markdown dividers anywhere in `README.md` — use `assets/divider.svg` instead.
- `WAKATIME_API_KEY` repo secret must be added manually by the user (GitHub Settings → Secrets → Actions) — no agent/tool here can create repo secrets.

---

### Task 1: Gradient divider SVG asset

**Files:**
- Create: `assets/divider.svg`

**Interfaces:**
- Produces: a repo-relative image at `./assets/divider.svg`, referenced by `README.md` in Tasks 2–4 as `<img src="./assets/divider.svg" width="100%" height="3" alt="" />`.

- [ ] **Step 1: Create the assets directory and the SVG file**

Write `assets/divider.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 3" width="1200" height="3" preserveAspectRatio="none">
  <defs>
    <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" stop-color="#00d4ff"/>
      <stop offset="50%" stop-color="#7b2ff7"/>
      <stop offset="100%" stop-color="#ff2d75"/>
    </linearGradient>
  </defs>
  <rect x="0" y="0" width="1200" height="3" rx="1.5" fill="url(#grad)"/>
</svg>
```

- [ ] **Step 2: Validate the SVG is well-formed XML**

Run: `xmllint --noout assets/divider.svg`
Expected: no output, exit code 0.

- [ ] **Step 3: Commit**

```bash
git add assets/divider.svg
git commit -m "feat: add gradient divider SVG asset"
```

---

### Task 2: README header — typing banner line

**Files:**
- Modify: `README.md:1-8` (banner comment, banner img, blank line, H1, intro paragraph, `---`, blank line)

**Interfaces:**
- Consumes: `assets/divider.svg` from Task 1.
- Produces: the header block later tasks append after (Task 3 starts immediately after this block's trailing divider).

- [ ] **Step 1: Replace the header block**

Old content (`README.md` lines 1-9):

```markdown
<!-- Banner: upload banner.png to this repo, or swap src for a hosted URL -->
<img src="./banner.png" alt="Nik Farees" width="100%" />

# Backend Developer — Laravel & Next.js

Backend developer working mainly in Laravel and Filament — that's what powers the management systems and admin platforms I build. On the frontend side I use Next.js when a project needs one. Between the two I've shipped 3+ management systems and a real-time auction platform serving 100+ concurrent users, plus the Docker setup and CI/CD pipelines behind them. Software Engineering grad from UniKL MIIT (CGPA 3.81).

---

### Stack
```

New content:

```markdown
<!-- Banner: upload banner.png to this repo, or swap src for a hosted URL -->
<img src="./banner.png" alt="Nik Farees" width="100%" />

<img src="https://readme-typing-svg.demolab.com/?font=Fira+Code&size=20&pause=1500&color=58A6FF&center=true&vCenter=true&width=600&lines=Backend+Developer;Laravel+%26+Filament;Next.js+when+needed;Auction+platform+-+100%2B+concurrent+users" alt="Typing SVG" />

# Backend Developer — Laravel & Next.js

Backend developer working mainly in Laravel and Filament — that's what powers the management systems and admin platforms I build. On the frontend side I use Next.js when a project needs one. Between the two I've shipped 3+ management systems and a real-time auction platform serving 100+ concurrent users, plus the Docker setup and CI/CD pipelines behind them. Software Engineering grad from UniKL MIIT (CGPA 3.81).

<img src="./assets/divider.svg" width="100%" height="3" alt="" />

### Stack
```

Use the Edit tool with the old/new blocks above (old_string is the exact 9 lines currently in the file; new_string is the replacement).

- [ ] **Step 2: Verify no `---` divider remains in the edited range and the typing URL is present**

Run: `grep -n "^---$" README.md | head -5; grep -c "readme-typing-svg" README.md`
Expected: first grep still shows 2 matches (Tasks 3 and 4 haven't run yet, so the two lower `---` lines are still there) — this step only confirms the *top* `---` (previously at line 8) is gone and the typing image line exists. Confirm by running `sed -n '1,10p' README.md` and visually checking there is no bare `---` line in that range and the `readme-typing-svg.demolab.com` URL appears once.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "feat: add typing-effect banner line, swap top divider for gradient SVG"
```

---

### Task 3: README — collapsible Stack section

**Files:**
- Modify: `README.md` (the `### Stack` section through the second `---`, i.e. the "Languages" / "Backend" / "Frontend" / "DevOps" / "Database" badge groups)

**Interfaces:**
- Consumes: `assets/divider.svg` from Task 1; runs immediately after Task 2's header block.
- Produces: closes with a gradient divider, which Task 4's Live Stats section starts right after.

- [ ] **Step 1: Replace the Stack section**

Old content (everything from `### Stack` through the `---` before `### Contact`, i.e. the current `README.md:10-46`):

```markdown
### Stack

**Languages**

![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![C#](https://img.shields.io/badge/C%23-512BD4?style=for-the-badge&logo=csharp&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

**Backend**

![Laravel](https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![Filament](https://img.shields.io/badge/Filament-FDAE4B?style=for-the-badge&logo=laravel&logoColor=black)
![REST APIs](https://img.shields.io/badge/REST%20APIs-005571?style=for-the-badge&logo=fastapi&logoColor=white)

**Frontend**

![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)

**DevOps**

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)

**Database**

![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white)

---

### Contact
```

New content:

```markdown
### Stack

<details>
<summary><strong>Languages</strong></summary>
<br>

![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![C#](https://img.shields.io/badge/C%23-512BD4?style=for-the-badge&logo=csharp&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

</details>

<details>
<summary><strong>Backend</strong></summary>
<br>

![Laravel](https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![Filament](https://img.shields.io/badge/Filament-FDAE4B?style=for-the-badge&logo=laravel&logoColor=black)
![REST APIs](https://img.shields.io/badge/REST%20APIs-005571?style=for-the-badge&logo=fastapi&logoColor=white)

</details>

<details>
<summary><strong>Frontend</strong></summary>
<br>

![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)

</details>

<details>
<summary><strong>DevOps</strong></summary>
<br>

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)

</details>

<details>
<summary><strong>Database</strong></summary>
<br>

![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white)

</details>

<img src="./assets/divider.svg" width="100%" height="3" alt="" />

### Contact
```

Use the Edit tool with the old/new blocks above.

- [ ] **Step 2: Verify all five `<details>` blocks are balanced**

Run: `grep -c "<details>" README.md; grep -c "</details>" README.md`
Expected: both print `5`.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "feat: collapse Stack badges into per-category <details> sections"
```

---

### Task 4: README — Live Stats section

**Files:**
- Modify: `README.md` (insert a new `### Live Stats` section between the Stack section's closing divider from Task 3 and `### Contact`)

**Interfaces:**
- Consumes: `assets/divider.svg` from Task 1; the `output` branch SVGs produced by `snake.yml` (Task 5); the `<!--START_SECTION:waka--> <!--END_SECTION:waka-->` markers filled by `waka-readme.yml` (Task 6).
- Produces: the waka marker comment pair that Task 6's workflow searches for verbatim — must be exactly `<!--START_SECTION:waka-->` and `<!--END_SECTION:waka-->`, each alone on its own line.

- [ ] **Step 1: Insert the Live Stats section before `### Contact`**

Old content (end of `README.md`, the divider + Contact section left after Task 3):

```markdown
<img src="./assets/divider.svg" width="100%" height="3" alt="" />

### Contact
```

New content:

```markdown
<img src="./assets/divider.svg" width="100%" height="3" alt="" />

### Live Stats

<table>
<tr>
<td valign="top" width="50%">

<img src="https://github-readme-stats.vercel.app/api?username=NikFarees&show_icons=true&theme=github_dark&hide_border=true&count_private=true" alt="GitHub Stats" width="100%" />

</td>
<td valign="top" width="50%">

<img src="https://github-readme-streak-stats.herokuapp.com/?user=NikFarees&theme=github-dark-blue&hide_border=true" alt="GitHub Streak" width="100%" />

</td>
</tr>
</table>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/NikFarees/NikFarees/output/github-contribution-grid-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/NikFarees/NikFarees/output/github-contribution-grid-snake.svg" />
  <img alt="contribution snake animation" src="https://raw.githubusercontent.com/NikFarees/NikFarees/output/github-contribution-grid-snake.svg" width="100%" />
</picture>

<!--START_SECTION:waka-->
<!--END_SECTION:waka-->

<img src="./assets/divider.svg" width="100%" height="3" alt="" />

### Contact
```

Use the Edit tool with the old/new blocks above.

- [ ] **Step 2: Verify the waka markers and snake picture tag are present and balanced**

Run: `grep -c "START_SECTION:waka" README.md; grep -c "END_SECTION:waka" README.md; grep -c "<picture>" README.md; grep -c "</picture>" README.md`
Expected: all four print `1`.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "feat: add Live Stats section (stats card, streak, snake, wakatime placeholder)"
```

---

### Task 5: `snake.yml` — contribution snake workflow

**Files:**
- Create: `.github/workflows/snake.yml`

**Interfaces:**
- Produces: `output` branch containing `github-contribution-grid-snake.svg` and `github-contribution-grid-snake-dark.svg`, filenames matching the `<picture>` tag added in Task 4.

- [ ] **Step 1: Create the workflow file**

Write `.github/workflows/snake.yml`:

```yaml
name: Generate Snake Animation

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  generate:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      - name: Generate snake SVGs
        uses: Platane/snk@v3
        with:
          github_user_name: NikFarees
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      - name: Push to output branch
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 2: Validate the workflow YAML parses**

Run: `ruby -ryaml -e "YAML.load_file('.github/workflows/snake.yml'); puts 'OK'"`
Expected: prints `OK`.

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/snake.yml
git commit -m "ci: add contribution-snake generation workflow"
```

---

### Task 6: `waka-readme.yml` — WakaTime stats workflow

**Files:**
- Create: `.github/workflows/waka-readme.yml`

**Interfaces:**
- Consumes: repo secret `WAKATIME_API_KEY` (user-provided, not created by this task); the `<!--START_SECTION:waka--> <!--END_SECTION:waka-->` markers from Task 4.
- Produces: commits to `main` updating the content between those markers.

- [ ] **Step 1: Create the workflow file**

Write `.github/workflows/waka-readme.yml`:

```yaml
name: Update WakaTime Stats

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

jobs:
  update-readme:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: anmol098/waka-readme-stats@master
        with:
          WAKATIME_API_KEY: ${{ secrets.WAKATIME_API_KEY }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SHOW_LANGUAGE: "True"
          SHOW_OS: "True"
          SHOW_TIMEZONE: "True"
          SHOW_PROJECTS: "True"
          SHOW_LINES_OF_CODE: "False"
          SHOW_LOC_CHART: "False"
          SHOW_SHORT_INFO: "True"
          SHOW_PROFILE_VIEWS: "False"
          SHOW_COMMIT: "False"
          SHOW_DAYS_OF_WEEK: "True"
          SHOW_UPDATED_DATE: "True"
          SYMBOL_VERSION: "1"
```

- [ ] **Step 2: Validate the workflow YAML parses**

Run: `ruby -ryaml -e "YAML.load_file('.github/workflows/waka-readme.yml'); puts 'OK'"`
Expected: prints `OK`.

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/waka-readme.yml
git commit -m "ci: add wakatime stats workflow"
```

---

### Task 7: Push, add secret, trigger workflows, verify live

**Files:** none (operational task — no file changes)

**Interfaces:**
- Consumes: all prior tasks' commits.
- Produces: a live, verified README on github.com and a populated `output` branch.

- [ ] **Step 1: Push `main` to origin**

Run: `git push origin main`
Expected: push succeeds (fast-forward, no conflicts — this repo has no other branches per `git status` at plan time).

- [ ] **Step 2: User adds the WakaTime secret**

Tell the user: go to `https://github.com/NikFarees/NikFarees/settings/secrets/actions`, click "New repository secret", name it `WAKATIME_API_KEY`, paste the key from wakatime.com/settings/api-key, save. This step cannot be automated — no tool here can write repo secrets.

- [ ] **Step 3: Manually trigger both workflows**

Run:
```bash
gh workflow run snake.yml --repo NikFarees/NikFarees
gh workflow run waka-readme.yml --repo NikFarees/NikFarees
```
Expected: both commands print a message confirming the workflow was triggered (no immediate output beyond that — `gh workflow run` is fire-and-forget).

- [ ] **Step 4: Watch both runs to completion**

Run: `gh run watch --repo NikFarees/NikFarees $(gh run list --repo NikFarees/NikFarees --workflow=snake.yml --limit=1 --json databaseId --jq '.[0].databaseId')`

Then repeat for waka-readme.yml: `gh run watch --repo NikFarees/NikFarees $(gh run list --repo NikFarees/NikFarees --workflow=waka-readme.yml --limit=1 --json databaseId --jq '.[0].databaseId')`

Expected: both report `✓ completed successfully`. If either fails, run `gh run view --repo NikFarees/NikFarees --log-failed` on that run's ID to see why (common causes: missing/invalid `WAKATIME_API_KEY`, `Platane/snk` action version mismatch) and fix before proceeding.

- [ ] **Step 5: Verify the `output` branch has the snake SVGs**

Run: `git fetch origin output && git show origin/output:github-contribution-grid-snake.svg | head -c 200`
Expected: prints the start of an `<svg ...>` document, not a 404/error.

- [ ] **Step 6: Verify the README renders correctly on GitHub**

Open `https://github.com/NikFarees/NikFarees` in a browser (or `mcp__claude-in-chrome__navigate` if driving this from the agent) and confirm: typing line animates, all five Stack `<details>` are collapsed by default and expand on click, no bare `---` line appears anywhere, GitHub Stats + Streak cards load, snake animation renders, WakaTime block shows real data (not empty markers), Contact badges unchanged.

This is the final acceptance check for the whole plan — no commit needed for this task.
