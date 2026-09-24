# Known issues

## 1. Certificate images not visible — RESOLVED

**Symptom:** The badge images in the Certificates section showed as broken, and the
"Certificate" links went nowhere.

**Root cause:** The asset files were moved into subfolders, but `index.html` still
pointed at the old top-level paths.

- The PNG badges were moved into `images/` and the PDFs into `certificates/`.
- `index.html` still referenced them as `src="basics-of-quantum-information.png"`,
  `href="IBMDesign20260924-22-gotr9u.pdf"`, and so on, relative to the repo root.
- The browser therefore requested files that no longer existed at those paths.

Nothing in the HTML or CSS was wrong. Only the file locations and the references
disagreed.

**Fix:**

- Moved the files with `git mv`, so git records them as renames rather than as
  deletions plus new files:
  - `*.png` → `images/`
  - `*.pdf` → `certificates/`
- Updated every reference in `index.html`:
  - `src="images/<name>.png"`
  - `href="certificates/<name>.pdf"`

**Verified:** I served the site over HTTP, the way GitHub Pages serves it. All 7
images loaded at full resolution, and all 7 PDF links returned HTTP 200.

**How to avoid it happening again:**

1. **Move files with `git mv old new`,** not in File Explorer, and then update
   `index.html` in the same commit. A move without a path update is exactly what
   caused this issue.
2. **Keep paths relative,** with no leading slash: `images/x.png`, not
   `/images/x.png`. The live site is served from a sub-path
   (`https://recursion2612.github.io/website-repo/`). A leading `/` would resolve
   to `recursion2612.github.io/images/…`, which is a 404.
3. **Match filename case exactly.** Windows ignores case, but GitHub Pages does
   not. `Images/Badge.PNG` works locally and breaks online if the file is
   `images/badge.png`.
4. **Check the links before committing.** Run this from the repo root in Git Bash.
   It prints `MISSING` for any referenced PNG or PDF that doesn't exist:

   ```bash
   for f in $(grep -oE '(src|href)="[^"#:]+\.(png|pdf)"' index.html | sed -E 's/.*="([^"]+)"/\1/' | sort -u); do [ -f "$f" ] || echo "MISSING $f"; done
   ```

   The template path `notes/your-file.pdf` also shows up, because the check reads
   HTML comments too. You can ignore it.

## 2. Images don't show in the Claude app's preview pane — limitation, not a bug

When you open `index.html` directly in the Claude desktop app's browser pane, it
renders the page as a static snapshot (a `data:` URL). A snapshot has no folder to
resolve relative paths against, so **no** local images load, even when the paths
are correct.

**Workaround:** Preview the site in one of these ways:

- Open `index.html` in a normal browser (double-click it, or use a `file:///` URL).
- Serve the folder over HTTP.
- Check the live GitHub Pages site after pushing.

## 3. Uncommitted edits in the main checkout — OPEN

The main checkout (`C:\Users\aasha\codes\website-repo`, branch `main`) has
uncommitted work: the manual file move from issue 1, plus some `index.html` edits.
Those edits contain three bugs:

| Where | Problem | Fix |
|---|---|---|
| Header contact link | `href="aashaypandharpatte@gmail.com"` has no `mailto:`, so the browser treats it as a relative page link and it 404s. | `href="mailto:aashaypandharpatte@gmail.com"` |
| Intro paragraph | Typo: "simu lation" | "simulation" |
| `<title>` | There's a line break before `</title>`. | Keep the title on one line. |

**Before merging this branch into `main`:**

- The manual copies in `main`'s untracked `images/` and `certificates/` folders are
  byte-identical to the files this branch adds. However, git refuses to merge over
  untracked files.
- In the main checkout, remove those two untracked folders and restore the deleted
  top-level files (`git restore .`). Then merge.
- Re-apply the email change, with the `mailto:` fix above, after merging.
- `git restore .` also discards the `index.html` edits in `main`. If you want to keep
  them, commit them first and resolve the conflict with this branch.
