# Preparing a repository for Librarian OS

This guide is for the person, or the AI agent, who adapts a repository so that Librarian OS can open it. It covers the `librarian-os.json` file, the layout of the content, the optional sidecar files that describe relations between notes, and a checklist. Follow it top to bottom for a new repository, or read the checklist at the end for an existing one.

## What Librarian OS needs

A repository is a Git repository (or any directory) with Markdown files. Librarian OS needs one thing from it: a file named `librarian-os.json` at the root that says what the project is and where its content starts. Everything else is optional. A repository with only that file and some Markdown opens and works. Sidecar files add typed relations, sources and practice questions on top.

```text
my-repo/
  librarian-os.json        required: project name and entry points
  docs/                    an entry point
    README.md              required: the entry page of the entry point
    getting-started.md
    getting-started.md.json   optional: sidecar (relations, sources, questions)
    reference/
      README.md            optional: an index for a subdirectory
      api.md
  notes/                   another entry point
    README.md
```

## Step 1: write librarian-os.json

```json
{
  "version": 1,
  "project": {
    "name": "My Knowledge Base",
    "description": "One or two sentences that appear on the start page.",
    "language": "en",
    "homepage": "https://example.com"
  },
  "entryPoints": [
    { "path": "docs" },
    { "path": "notes", "title": "Field notes", "description": "Shown on the start page instead of the README summary." }
  ],
  "ignore": ["drafts", "_archive"],
  "toc": { "file": "ToC.md", "check": false, "descriptions": { "Makefile": "Build commands." } },
  "rules": { "requireSidecars": false, "noLongDashes": false }
}
```

### Fields

| Field | Required | Meaning |
| --- | --- | --- |
| `version` | yes | The version of this standard. Always `1`. |
| `project.name` | yes | Shown next to "Librarian OS" in the top bar, in the footer, in page titles and on the start page. Up to 80 characters. |
| `project.description` | no | Shown on the start page. Up to 400 characters. |
| `project.language` | no | Language code of the content, for example `en` or `pl`. Default `en`. |
| `project.homepage` | no | An `http` or `https` address. |
| `entryPoints` | yes | At least one. Each entry is a directory. The reader shows these directories as the top level of the tree, in the order given. |
| `entryPoints[].path` | yes | Directory relative to the repository root, with forward slashes. No leading slash, no `..`, no directory that starts with a dot. Entry points must not contain each other. |
| `entryPoints[].title` | no | Replaces the title taken from the README heading. |
| `entryPoints[].description` | no | Replaces the description taken from the README summary. |
| `ignore` | no | Directory names or repository relative paths that are skipped inside the entry points. `node_modules`, `dist` and directories that start with a dot are always skipped. |
| `toc.file` | no | Name of the table of contents file that `npm run toc` writes. Default `ToC.md`. |
| `toc.check` | no | When `true`, `npm run check` verifies that the table of contents lists every file. Default `false`. |
| `toc.descriptions` | no | Descriptions for files that are not Markdown notes, keyed by path. Used by `npm run toc`. |
| `rules.requireSidecars` | no | When `true`, every Markdown file in an entry point must have a sidecar. Default `false`. |
| `rules.noLongDashes` | no | When `true`, `npm run check` reports em dashes and en dashes in text files. Default `false`. |

Things outside the entry points are ignored by the reader. That includes a root `README.md`, license files, build files and any directory you do not list. They are not shown in the tree.

## Step 2: make every entry point valid

1. The entry point directory has a `README.md`. It is the page that opens when someone clicks the entry point. Its first heading becomes the title in the tree.
2. Every Markdown file has exactly one `#` heading, which is its title. Use `##` and `###` below it without skipping levels.
3. Relative links between files work (`[text](../other/file.md#heading)`). The reader turns links to Markdown files into in-app navigation. Images next to the notes are shown too (`png`, `jpg`, `gif`, `webp`, `svg`, `pdf`).
4. A directory with a `README.md` is shown as a section with its files below it. A directory without one is not a node, and its files are attached to the nearest parent that has a README.
5. A paragraph that starts with `TODO:` is highlighted as an open gap, and the count is shown in the note header.
6. A `mermaid` code fence becomes a diagram. Math between `$$ ... $$` is rendered when present.

## Step 3 (optional): add sidecars

A sidecar is a JSON file with the same name as the Markdown file plus `.json`: `note.md` and `note.md.json`, `README.md` and `README.md.json`. It holds what Markdown cannot express well. The Markdown file stays the single source of truth for the text.

```json
{
  "node": {
    "id": "docs.getting-started",
    "kind": "note",
    "status": "draft",
    "level": "intro",
    "order": 1,
    "tags": ["setup"],
    "aliases": ["quick start"]
  },
  "links": [
    { "to": "reference.api", "rel": "prerequisite", "note": "Read the API overview first" }
  ],
  "sources": [
    { "key": "rfc-9110", "type": "standard", "title": "HTTP Semantics", "url": "https://www.rfc-editor.org/rfc/rfc9110", "verified": true, "checked": "2026-10" }
  ],
  "concepts": [
    { "term": "Idempotent", "anchor": "#methods", "definition": "Repeating the request has the same effect as sending it once." }
  ],
  "questions": [
    { "id": "q1", "type": "explain", "q": "Why is PUT idempotent?", "a": "Because repeating it leaves the resource in the same state.", "anchor": "#methods", "origin": "author" }
  ],
  "map": { "group": "setup" }
}
```

### Sidecar fields

| Field | Required | Meaning |
| --- | --- | --- |
| `node.id` | yes | Stable identifier, unique in the repository: lowercase letters, digits and hyphens, with segments joined by dots. Links use ids, so moving a file does not break them. Without a sidecar the id is derived from the path. |
| `node.kind` | yes | `note` for a topic file, `index` for a `README.md`. |
| `node.status` | no | `draft` or `reviewed`. |
| `node.level` | no | `intro`, `intermediate` or `advanced`. |
| `node.order` | no | Position among siblings in the tree. Files without it come last, sorted by path. |
| `node.tags`, `node.aliases` | no | Used for filtering and search. |
| `links` | no | Typed relations from this note to others. |
| `sources` | no | `key`, `type` (`paper`, `book`, `report`, `standard`, `documentation`, `website`, `course`, `other`), `title`, and optionally `authors`, `year`, `publisher`, `url`, `verified`, `checked` (`YYYY-MM`). |
| `concepts` | no | Key terms with an `anchor` to the heading that explains them and a one sentence `definition`. |
| `questions` | no | Practice questions: `id`, `type` (`recall`, `explain`, `apply`, `compare`), `q`, `a`, and optionally `hint`, `anchor`, `origin` (`author` or `ai`), `reviewed`. |
| `map.group` | no | A label for grouping in mind maps. |

### Relation types

| `rel` | Meaning | Shown on the target as |
| --- | --- | --- |
| `prerequisite` | Read the target first. | Required by |
| `extends` | Builds on or deepens the target. | Extended by |
| `related` | Neighbors in the same topic, no order. | Related |
| `contrasts` | An alternative or opposing view. | Contrasts |
| `example-of` | A concrete case of the target. | Examples |
| `applies` | Applies the target idea in a specific setting. | Applied in |
| `part-of` | A part of a larger topic note. | Parts |

### Rules for sidecars

- Relations point forward only. Backlinks are computed, so never write the inverse relation in the target.
- A relation target must be an existing `node.id`. A missing target is an error.
- The title comes from the `#` heading and the hierarchy from the directory tree. Do not store them again.
- Use UTF-8, two spaces for indentation, one trailing newline, no comments.
- `anchor` values are the GitHub style slugs of headings: lowercase, punctuation removed, spaces turned into hyphens.
- Create missing sidecars with `npm run scaffold`, then add relations and sources by hand.

## Step 4: check and run

From the `librarian-os` directory (the repository path is the last argument; without it, the directory where npm was started is used):

```bash
npm run check -- /absolute/path/to/my-repo      # validate configuration, sidecars, links
npm run scaffold -- /absolute/path/to/my-repo   # create missing sidecars (--dry to preview)
npm run toc -- /absolute/path/to/my-repo        # write the table of contents file
npm run dev -- --repo /absolute/path/to/my-repo # open it in the reader
```

`npm run check` exits with code 1 when it finds errors. It checks `librarian-os.json` against its schema, that each entry point has a README, sidecar schema and format, duplicate ids, unknown relation targets, kind and path agreement, one title per file, broken relative links and anchors, and, when switched on, the table of contents and the dash rule.

## Adding the repository in the reader

In the reader, open **Settings, Repositories**. Paste an absolute path to a local directory, or an `https://` or `ssh` Git URL. A URL is cloned (shallow) into the server data directory, and later additions of the same URL update the copy with a fast-forward pull. The list is stored on the server, and the top bar has a switcher for it. Removing a repository from the list never deletes files.

## Getting your changes into the reader

The reader keeps the loaded repository in the browser, so a change on disk or in the remote needs a reload. Press **Refresh** in the top bar. For a git working copy it runs `git pull --ff-only` first (a diverged copy is reported, not merged), then reloads the notes. A repository that was added by URL is also pulled when you switch to it. In **Settings, Repositories**, each repository has a "Pull and refresh" button too.

## Checklist for adapting an existing repository

1. Decide the entry points: the directories that hold the knowledge. Leave build files, vendored code and generated output out.
2. Create `librarian-os.json` with a clear `project.name` and `description`.
3. Add a `README.md` to every entry point, with one `#` heading and a short first paragraph (it becomes the summary).
4. Make sure every Markdown file has exactly one `#` heading.
5. Fix broken relative links (`npm run check` lists them with file and line).
6. Optionally run `npm run scaffold`, then add `order`, `tags` and `links` where they help.
7. Optionally switch on `rules.requireSidecars`, `toc.check` or `rules.noLongDashes` if the repository wants those rules.
8. Run `npm run check` until it reports `0 errors`.
9. Open the repository in the reader and look at the tree, a note, the graph and the search.

## Instructions for an AI agent

If you are an AI agent asked to prepare a repository, do this in order and report what you did.

1. Read this file and the repository's own guidance (`README.md`, `CLAUDE.md`, `AGENTS.md` or similar) first. The repository's rules win over defaults here.
2. List the directories with Markdown content and propose entry points. Ask when two structures are plausible.
3. Write `librarian-os.json`. Take the project name and description from the repository itself. Do not invent a description.
4. Add missing entry point READMEs and fix headings and links, changing as little prose as possible.
5. Run `npm run scaffold`, then relations and sources only where the text states them. Never fabricate a source, a url or a `verified` flag.
6. Run `npm run check` and fix every error. Do not silence a check by switching off a rule that the repository did not ask to switch off.
7. Report: the entry points chosen, files created or changed, the output of `npm run check`, and anything left undecided.

## Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| "librarian-os.json not found" | The file must be at the repository root, not in a subdirectory. |
| "entry point ... has no README.md" | Add a `README.md` to that directory. |
| A note is missing from the tree | It is outside every entry point, or its directory is in `ignore`. |
| A note appears as a top level item | It has no README in its directory and sits directly in an entry point. Add a README, or move it. |
| "link to unknown id" | The `to` value is not the `node.id` of any note. Check for typos and for notes that have no sidecar yet. |
| Cloning a URL fails | Only `https://` and `ssh` URLs are accepted. Private repositories need credentials that Git can use without a prompt, for example an SSH key. |

## Roadmaps and local progress

A repository may provide one or more `_roadmaps/*.json` files. Each file follows [the roadmap schema](schema/roadmap.schema.json): `version`, stable `id`, `title`, `nodes`, and `edges`. A node has a stable `id`, `title`, optional `group` and `description`, `documentation` paths relative to the repository root, and `externalSources` as HTTPS URLs. An edge has `from`, `to`, and `type` (`next` or `branch`). The graph must be acyclic. A terminal node may use `targetRoadmap` to link to another roadmap ID in the same repository. Redirect nodes cannot have outgoing edges. All documentation paths must name Markdown files under the repository's configured entry points. Run `npm run check -- /path/to/repo` to validate roadmap files.

The reader shows roadmap progress separately from reading status. Both are set manually. Local preferences, completed roadmap nodes, and read article fingerprints are saved under the server's `dataDir/user-state/`, one file per registered repository. When an article changes, it becomes unread and the next refresh lists changed articles. These local state files are not written to the content repository.
