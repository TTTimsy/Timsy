# Yellow Contribution Animation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the profile README snake animation with a light-yellow animated contribution graph inspired by `Man0dya/Readme-Contribution-Graph-Generator`.

**Architecture:** The profile repo stores a focused Node script that queries GitHub GraphQL contribution data and writes animated SVG files into the repository root. A GitHub Actions workflow runs that script on push, manual dispatch, and daily schedule, then commits changed SVGs. The README embeds the generated light/dark SVGs as the profile cover.

**Tech Stack:** GitHub Actions, Node.js 20, GitHub GraphQL API, SVG SMIL animation, GitHub REST Contents API for remote edits.

---

## File Structure

- Create `scripts/generate-svg.cjs`: single-purpose generator that fetches public contribution calendar data and writes light/dark animated SVG files.
- Create `.github/workflows/generate-contribution-animation.yml`: workflow that runs the generator and auto-commits SVG outputs.
- Modify `README.md`: profile cover using generated SVG files; no snake references.
- Delete `.github/workflows/snake.yml`: removes old snake generation.
- Create `docs/superpowers/plans/2026-05-11-yellow-contribution-animation.md`: this implementation plan.

---

### Task 1: Remote State RED Checks

**Files:**
- Inspect: `README.md`
- Inspect: `.github/workflows/snake.yml`
- Inspect: `scripts/generate-svg.cjs`

- [ ] **Step 1: Run failing validation against current README**

Run:

```powershell
$readme = gh api "repos/TTTimsy/TTTimsy/contents/README.md?ref=main" --header "Accept: application/vnd.github.raw"
if ($readme -match 'snake|github-contribution-grid-snake|Platane') { exit 1 } else { exit 0 }
```

Expected: FAIL because README still references snake files.

- [ ] **Step 2: Run failing validation for missing generator script**

Run:

```powershell
gh api "repos/TTTimsy/TTTimsy/contents/scripts/generate-svg.cjs?ref=main"
```

Expected: FAIL with not found because the generator script does not exist yet.

### Task 2: Add Generator Script

**Files:**
- Create: `scripts/generate-svg.cjs`

- [ ] **Step 1: Add script with these behaviors**

The script must:

- Resolve username from `CONTRIBUTION_USERNAME` or `GITHUB_REPOSITORY` owner.
- Require `GITHUB_TOKEN`, `GH_TOKEN`, or `GITHUB_AUTH_TOKEN`.
- Query GitHub GraphQL `contributionsCollection.contributionCalendar.weeks`.
- Generate light and dark animated SVGs.
- Use yellow color palettes.
- Write these files: `TTTimsy-contribution-animation.svg`, `TTTimsy-contribution-animation-dark.svg`, `contribution-animation.svg`, `contribution-animation-dark.svg`, `github-contribution-animation.svg`, `github-contribution-animation-dark.svg`.

- [ ] **Step 2: Verify script content remotely**

Run:

```powershell
$script = gh api "repos/TTTimsy/TTTimsy/contents/scripts/generate-svg.cjs?ref=main" --header "Accept: application/vnd.github.raw"
$checks = @(
  $script -match 'fetchContributionData',
  $script -match 'buildAnimatedSvg',
  $script -match '#f7d774',
  $script -match 'TTTimsy-contribution-animation.svg'
)
if ($checks -contains $false) { exit 1 } else { exit 0 }
```

Expected: PASS.

### Task 3: Add Workflow and Remove Snake Workflow

**Files:**
- Create: `.github/workflows/generate-contribution-animation.yml`
- Delete: `.github/workflows/snake.yml`

- [ ] **Step 1: Add workflow**

The workflow must use Node 20, run `node scripts/generate-svg.cjs`, pass `GITHUB_TOKEN` and `CONTRIBUTION_USERNAME`, and auto-commit generated SVGs.

- [ ] **Step 2: Delete old snake workflow**

Delete `.github/workflows/snake.yml` using the GitHub Contents API with the current blob SHA.

- [ ] **Step 3: Verify workflow state**

Run:

```powershell
gh api "repos/TTTimsy/TTTimsy/contents/.github/workflows/generate-contribution-animation.yml?ref=main" --header "Accept: application/vnd.github.raw"
gh api "repos/TTTimsy/TTTimsy/contents/.github/workflows/snake.yml?ref=main"
```

Expected: new workflow exists; snake workflow returns not found.

### Task 4: Update README Cover

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Replace README content**

README should contain centered profile cover HTML with:

```html
<div align="center">

# Hi, I'm Timsy

<p>Exploring code, materials, and useful automation.</p>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="TTTimsy-contribution-animation-dark.svg" />
  <img alt="TTTimsy contribution animation" src="TTTimsy-contribution-animation.svg" width="100%" />
</picture>

<br />

<img src="https://capsule-render.vercel.app/api?type=rect&height=2&color=f7d774&section=footer" alt="divider" width="100%" />

</div>
```

- [ ] **Step 2: Verify README references**

Run:

```powershell
$readme = gh api "repos/TTTimsy/TTTimsy/contents/README.md?ref=main" --header "Accept: application/vnd.github.raw"
if (($readme -match 'TTTimsy-contribution-animation.svg') -and ($readme -match 'TTTimsy-contribution-animation-dark.svg') -and ($readme -notmatch 'snake|github-contribution-grid-snake|Platane')) { exit 0 } else { exit 1 }
```

Expected: PASS.

### Task 5: Generate and Verify Assets

**Files:**
- Generated by workflow: `TTTimsy-contribution-animation.svg`
- Generated by workflow: `TTTimsy-contribution-animation-dark.svg`
- Generated by workflow: `github-contribution-animation.svg`
- Generated by workflow: `github-contribution-animation-dark.svg`

- [ ] **Step 1: Run workflow**

Run:

```powershell
gh workflow run generate-contribution-animation.yml -R TTTimsy/TTTimsy
```

Expected: workflow dispatch accepted.

- [ ] **Step 2: Confirm latest workflow succeeds**

Run:

```powershell
gh run list -R TTTimsy/TTTimsy --workflow generate-contribution-animation.yml --limit 1
```

Expected: latest run is `completed success`.

- [ ] **Step 3: Confirm generated SVGs exist and animate**

Run:

```powershell
$svg = gh api "repos/TTTimsy/TTTimsy/contents/TTTimsy-contribution-animation.svg?ref=main" --header "Accept: application/vnd.github.raw"
if (($svg -match '<animate') -and ($svg -match '#f7d774|#ffd966|#d4a72c')) { exit 0 } else { exit 1 }
```

Expected: PASS.