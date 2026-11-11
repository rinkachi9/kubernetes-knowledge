# Guideline

This document describes how to approach the Kubernetes Knowledge repository: what belongs in it, how notes are organized, how concepts must be explained and what quality bar every note has to meet. It applies to people and to AI agents alike.

## Table of Contents

1. [Purpose and audience](#1-purpose-and-audience)
2. [Core principles](#2-core-principles)
3. [Repository structure and naming](#3-repository-structure-and-naming)
4. [Anatomy of a note](#4-anatomy-of-a-note)
5. [How to explain a concept](#5-how-to-explain-a-concept)
6. [Examples, code and diagrams](#6-examples-code-and-diagrams)
7. [Writing style](#7-writing-style)
8. [Sources, references and verification](#8-sources-references-and-verification)
9. [Rules specific to this repository](#9-rules-specific-to-this-repository)
10. [Quality checklist](#10-quality-checklist)
11. [Maintenance and versioning](#11-maintenance-and-versioning)
12. [Working with AI agents](#12-working-with-ai-agents)

## 1. Purpose and audience

The repository is a long term reference for Kubernetes: its architecture, its API objects and the mechanisms behind them, the way workloads are run, networked, stored, secured, scheduled and observed, and the practices and ecosystem around it. It is written at an expert level for platform engineers, DevOps engineers, SREs and architects. Notes explain upstream Kubernetes first and treat distributions, managed services and tools as examples. Manifests and commands illustrate an idea and are never the subject of the note. It has two jobs:

- **Teach.** Someone with general technical background but no experience in the specific topic should be able to read a note and come away with a correct mental model.
- **Recall.** Someone who already knows the topic should be able to find the exact detail they need quickly, without wading through introductions.

To serve both, every note is layered. It starts with a short plain language summary, then builds up to precise detail. A reader can stop at any layer and still have learned something true.

The primary audience is the future version of the author. Write as if explaining to a smart colleague who was not in the room when you learned the topic. Do not assume they remember the context, the tools you used or the problem you were solving at the time.

## 2. Core principles

**Depth over breadth.** A note that explains one idea thoroughly is worth more than five that name-drop ten. Cover a topic completely before moving on: what it is, why it exists, how it works, when to use it, when not to use it, and how it fails.

**Explain the why, not only the what.** Facts without reasons are trivia. For every rule, tool or technique, state the problem it solves. A reader who understands the problem can reconstruct the solution and can tell when it does not apply.

*Example.* "Use parameterized queries" is a rule. "Parameterized queries keep data and code separate, so input can never change the structure of the statement" is understanding.

**Precision.** Use the correct technical term, define it on first use and keep using it consistently. Do not swap synonyms for variety, because a reader may assume two words mean two things. Quantify when you can: "about 10 ms on a warm cache" beats "fast".

**Show, then generalize.** Introduce an abstraction with a concrete case first, or immediately after. People learn patterns from instances.

**Be honest about uncertainty.** If something is unverified, simplified or version dependent, say so in the note. A clearly marked gap is better than a confident error.

**One idea, one place.** Each concept has a single home note. Other notes link to it. Duplicated explanations drift apart and become contradictory.

**Make it verifiable.** Claims link to sources. Examples can be run or reproduced. Numbers say where they came from.

**Show it as well as say it.** Notes are meant to be attractive to read and quick to grasp. A structure, a process, a model or a set of numbers is drawn, not only described. Every note has at least one diagram, schema, model or chart that carries real information (see [section 6.2](#62-diagrams-models-and-visuals)).

## 3. Repository structure and naming

### 3.1 Layout

The structure follows the *Concepts* section of the official [Kubernetes documentation](https://kubernetes.io/docs/concepts/), which is the canonical, community maintained division of the domain. Each top level chapter of Concepts is an *area* (a top level directory such as `workloads/`), in the order of the documentation. Inside an area, a *section* groups notes that the documentation also groups (such as `workloads/pods/` for the pages under "Pods"). One note corresponds to one page of Concepts, or to one clearly separable part of a page. Checked against the Concepts navigation of the Kubernetes documentation v1.37 in November 2026. The list of areas is in the [README](README.md#areas).

Three areas are not chapters of Concepts, because the documentation does not cover their subject:

- `application-delivery/` holds the practices for getting applications into a cluster: CI/CD integration, GitOps, packaging and deployment strategies.
- `ecosystem/` holds products and projects around Kubernetes: managed services, distributions, alternative orchestrators and third party tools. Its sections follow the categories of the [CNCF Cloud Native Landscape](https://landscape.cncf.io/).
- `labs/` holds hands-on material: home lab setups and exercises that combine several areas.

```text
ToC.md                  # full tree of every file in the repository
_sources/               # original exports that notes are migrated from (not committed)
librarian-os.json       # configuration for Librarian OS: project name, entry points, rules (see 3.5)
_assets/
    <area>/             # images, diagram sources, media and other files used by notes
<area>/
    README.md           # entry point: what the area covers, reading order, planned topics
    README.md.json      # its sidecar
    <topic>.md          # a single note
    <topic>.md.json     # its sidecar: node header and relations (see 3.5)
    <section>/          # optional, groups related notes as the documentation does
        README.md
        <topic>.md
        <topic>/        # optional, when a topic has its own scripts or files
            README.md   # the note itself
            scripts/    # scripts that belong to this topic only
```

Guidance:

- **Areas and sections come from the documentation.** Every chapter of Concepts has an area, even when it has no notes yet (`resource-management/` and `windows/` start as planned areas). A page of Concepts that has no note yet is listed under "Planned topics" in the README of its area or section.
- **Prefer flat over deep.** Area and section are the normal depth. Add a section when the documentation groups pages (Pods, Workload Management, Autoscaling) or when an area has more than about twelve files.
- **One concept, one home.** A concept has one home note in the area where Concepts places it. Other areas link to it. For example, NetworkPolicy lives in `services-and-networking/` and the security overview links to it; LimitRange lives in `policies/` and the resource management note links to it; probes live in `workloads/pods/`.
- **Every area and section has a `README.md`.** It is the entry point. It lists its notes in a recommended reading order with a one line description each, and lists the planned topics. Areas without notes have a `README.md` that says so, lists the planned topics and contains a `TODO:` line.
- **Media goes to `_assets/<area>/`, scripts stay with their topic.** Images, diagram sources, downloadable files and media go to `_assets/<area>/`. A script or a set of manifests lives next to the material it belongs to: when a topic gets scripts, the note becomes a directory `<topic>/` with `README.md` (the note) and a `scripts/` directory. There is no global `scripts/` directory.
- **`librarian-os.json` lists the entry points.** The area directories are the entry points, in the order of Concepts followed by `application-delivery/`, `ecosystem/` and `labs/`. Adding an area means adding it to the `entryPoints` list in the same change. Content outside the entry points (this guideline, the README, `_sources/`, `_assets/`) is not shown as notes by Librarian OS.
- **Keep `ToC.md` complete.** See [section 3.4](#34-table-of-contents).

*Analogy.* Think of a library. Areas are wings, sections are the shelves in a wing, READMEs are the catalog cards for a shelf, and notes are the books. The catalog follows the official documentation, so a reader who knows where kubernetes.io puts a page knows where the note is. The analogy breaks where a book could stand on two shelves: here it stands on one, and the other shelf holds a card that points to it.

### 3.1.1 Placement of a note

Classify a note by the main question it answers and place it in the area whose Concepts chapter covers that question:

| Area directory | Primary home for |
| --- | --- |
| `overview/` | What Kubernetes is, its components at a glance, objects, names, labels, namespaces, annotations, the Kubernetes API and kubectl. |
| `cluster-architecture/` | Control plane and node components, etcd, nodes, controllers, leases, the cloud controller manager, cgroups, garbage collection. |
| `containers/` | Container fundamentals, images, container environment, RuntimeClass, lifecycle hooks, the CRI and container technologies such as Docker and LXC. |
| `workloads/` | Pods and their lifecycle, init, sidecar and ephemeral containers, probes, disruptions, QoS classes, workload controllers and workload autoscaling. |
| `services-and-networking/` | The network model, IP ranges and dual-stack, Services, Ingress, Gateway API, EndpointSlices, network policies and DNS. |
| `storage/` | Volumes, persistent volumes and claims, storage classes, provisioning, snapshots and CSI. |
| `configuration/` | Configuration good practices, ConfigMaps, Secrets, resource requests and limits, kubeconfig. |
| `security/` | Security overview, Pod Security Standards and admission, service accounts, access control, RBAC good practices, security context, multi-tenancy. |
| `policies/` | LimitRange, ResourceQuota and process ID limits. |
| `scheduling-preemption-and-eviction/` | kube-scheduler, the scheduling framework, node assignment, taints and tolerations, topology spread, priority and preemption, eviction. |
| `resource-management/` | Node resource managers and dynamic resource allocation. |
| `cluster-administration/` | Node autoscaling, certificates, cluster networking setup, observability of the cluster (logging, metrics, traces), API priority and fairness, addons. |
| `windows/` | Windows nodes and Windows containers. |
| `extending-kubernetes/` | Extension points, custom resources, the aggregation layer, operators, network and device plugins, scheduler extensions. |
| `application-delivery/` | CI/CD integration, GitOps, packaging with Helm and Kustomize, deployment strategies and rollbacks. |
| `ecosystem/` | Managed services, distributions, alternative orchestrators and third party tools, grouped by CNCF Landscape category. |
| `labs/` | Home lab setups and hands-on exercises that combine several areas. |

For a topic that fits several areas, choose the home by its central mechanism and learning goal, and link to it from the other places. Do not copy the explanation into several directories. When a note moves, keep its sidecar `node.id` so that relations stay valid, and update READMEs, relative links and `ToC.md`. When the Concepts navigation changes in a new Kubernetes release, compare it with this structure and document the changes in the README before renaming directories.

### 3.2 Naming

| Item | Convention | Example |
| --- | --- | --- |
| Directories | lowercase, words separated by hyphens | `services-and-networking/` |
| Note files | lowercase, hyphens, descriptive noun phrase, `.md` | `pod-topology-spread-constraints.md` |
| Images and media | note name, then a short description, stored in `_assets/<area>/` | `_assets/services-and-networking/ingress-simple-fanout.png` |
| Diagram sources | same base name as the rendered image, source format as extension | `_assets/services-and-networking/ingress-simple-fanout.drawio` |
| Scripts | lowercase, hyphens, verb phrase, stored in the `scripts/` directory of their topic | `labs/home-lab/scripts/create-kind-cluster.sh` |
| Entry points | always `README.md` | `<area>/<section>/README.md` |
| Table of contents | always `ToC.md` in the repository root | `ToC.md` |
| Sidecar | the full name of the Markdown file plus `.json` | `deployments.md.json`, `README.md.json` |

Names describe the content, not its status or origin. Avoid `notes.md`, `misc.md`, `new-draft-2.md` or dates in file names. Do not use spaces or uppercase letters, so that links and shell commands behave the same on every system.

### 3.3 Links

- Use relative links between notes, for example `[indexes](../query-performance/b-tree-indexes.md)`.
- Link to a heading when you point at a specific part: `[isolation levels](transactions.md#isolation-levels)`.
- Link text describes the target ("the section on isolation levels"), never "click here".
- When a prerequisite is needed, link it in the note's front section instead of re-explaining it.

### 3.4 Table of contents

`ToC.md` in the repository root is the complete map of the repository. It is an indented tree with one entry for every file: root documents, area and section `README.md` files, notes, assets and scripts. Each entry is a relative link followed by a short description. A sidecar (`.md.json`) is not listed on its own: the entry of its Markdown file covers it. 

- **Every change is reflected in `ToC.md`.** Adding, moving, renaming or deleting a note, an asset or a script updates `ToC.md` in the same change. Run `npm run toc` in Librarian OS with this repository as the argument to regenerate it: entries come from the files, the note titles and summaries, and the "Planned topics" lists of the READMEs, so a change of title or summary updates the description there.
- **Planned areas are marked.** An area or section that has no notes is marked `(planned)`. Planned topics are listed without a link.
- **Verify before finishing.** Compare `ToC.md` with the output of `git ls-files --cached --others --exclude-standard` and check that every link resolves.
- **`ToC.md` is not a replacement for section READMEs.** The section README keeps the recommended reading order and the context. `ToC.md` keeps the complete list.

### 3.5 Sidecar metadata

Every Markdown file inside an entry point (a directory listed in `librarian-os.json`) has a *sidecar*: a JSON file with the same name plus `.json`, stored next to it (`deployments.md` and `deployments.md.json`, `README.md` and `README.md.json`). The Markdown file stays the single source of truth for the text. The sidecar holds only what Markdown cannot express well: the identity of the note as a node in a graph, typed relations to other notes, structured sources, a list of key terms and practice questions. The Librarian OS reader (the repository is configured for it by [`librarian-os.json`](librarian-os.json)) uses the sidecars to draw mind maps, show backlinks, search and build practice decks. AI tools use them to generate questions without parsing prose.

*Analogy.* The note is a book on the shelf, and the sidecar is its catalog card. The card does not repeat the book. It says what the book is, what it needs to be read after and what it connects to. The limit of the analogy is that a catalog card is written once, while a sidecar must be kept in step with the note when the note changes.

**Example** (`workloads/workload-management/deployments.md.json`, shortened):

```json
{
  "node": {
    "id": "workloads.workload-management.deployments",
    "kind": "note",
    "status": "draft",
    "level": "intermediate",
    "order": 1,
    "tags": ["workloads", "workload-management"],
    "aliases": ["deployment controller"]
  },
  "links": [
    { "to": "workloads.workload-management.replicaset", "rel": "prerequisite" }
  ],
  "sources": [
    {
      "key": "k8s-deployments",
      "type": "documentation",
      "publisher": "Kubernetes documentation",
      "title": "Deployments",
      "url": "https://kubernetes.io/docs/concepts/workloads/controllers/deployment/",
      "verified": false
    }
  ],
  "concepts": [
    { "term": "Proportional scaling", "anchor": "#proportional-scaling", "definition": "Spreading additional replicas across the active ReplicaSets of a rollout in proportion to their size." }
  ],
  "questions": [
    { "id": "q1", "type": "explain", "q": "Why does scaling a Deployment not create a new revision?", "a": "Because a revision is created only when the Pod template changes.", "anchor": "#rolling-back", "origin": "author" }
  ],
  "map": { "group": "workload-management" }
}
```

In this repository the `node.id` is the path of the file without the extension, with `/` replaced by `.` (for a README, the path of its directory). The id is kept when a file moves.

**Fields**

| Field | Required | Meaning |
| --- | --- | --- |
| `$schema` | no | Path or URL of the JSON Schema of Librarian OS (`schema/note.schema.json`), so that editors can validate and autocomplete. |
| `node.id` | yes | Stable identifier, unique in the repository: lowercase segments of letters, digits and hyphens joined by dots. Links use ids, not paths, so moving a file does not break them. |
| `node.kind` | yes | `note` (a topic note), or `index` (an area or section README). Documents at the repository root, such as this one, are outside the entry points and have no sidecar. |
| `node.status` | no | `draft` or `reviewed`. Default is `draft`. |
| `node.level` | no | `intro`, `intermediate` or `advanced`. |
| `node.tags`, `node.aliases` | no | Free tags for filtering, and alternative names used by search. |
| `links` | no | Typed relations from this note to others (table below). |
| `sources` | no | Structured bibliography: `key`, `type`, `authors`, `year`, `title`, `url`, `verified`, `checked`. The human readable list stays in "Further reading". |
| `concepts` | no | Key terms with an `anchor` to the heading that explains them and a one sentence `definition`. |
| `questions` | no | Practice questions with `id`, `type` (`recall`, `explain`, `apply`, `compare`), `q`, `a`, optional `hint`, `anchor`, `origin` (`author` or `ai`) and `reviewed`. |
| `map` | no | Hints for mind maps, such as `group`. |

**Relation types**

| `rel` | Meaning | Shown on the target as |
| --- | --- | --- |
| `prerequisite` | Read the target first. | Required by |
| `extends` | Builds on or deepens the target. | Extended by |
| `related` | Neighbors in the same topic, no order. | Related |
| `contrasts` | An alternative or opposing view. | Contrasts |
| `example-of` | A concrete case of the target. | Examples |
| `applies` | Applies the target idea in a specific setting. | Applied in |
| `part-of` | A part of a larger topic note. | Parts |

**Rules**

- **Follow `INSTRUCTIONS.md` of Librarian OS.** Whenever you create or change a note, its sidecar, a README or `librarian-os.json`, follow `INSTRUCTIONS.md` in the Librarian OS repository for everything that concerns compatibility with the reader: the configuration standard, the sidecar standard, entry points and the steps to adapt a repository. If this guideline and `INSTRUCTIONS.md` disagree on a Librarian OS matter, `INSTRUCTIONS.md` wins for the technical format, and this guideline wins for content and style.
- **Derived, not stored.** The title comes from the `#` heading and the area and sections come from the path. The hierarchy edges of the graph come from the directory tree. Storing them again would let them drift.
- **Relations point forward only.** Backlinks are computed. Do not write the inverse relation in the target.
- **Link only to existing ids.** A planned topic without a sidecar cannot be a target. `npm run check` fails on a broken id.
- **No duplicated prose.** A definition in `concepts` is one sentence. The explanation belongs in the note.
- **Practice questions.** The "Practice" section of the note stays the authoritative place for questions the author writes. The `questions` list is the deck for the reader and for generated questions, so do not copy the same question into both. Questions with `origin: "ai"` start with `reviewed: false` until the author accepts them.
- **Format.** UTF-8, 2 space indent, one trailing newline, no comments, English only. The hyphen rule of [section 7.2](#72-punctuation-hyphens-only) applies to every string.
- **Validation.** `npm run check` in Librarian OS (with this repository as the argument) validates every sidecar against the schema and checks ids, relation targets, titles, links and `ToC.md`. `npm run scaffold` creates missing sidecars.

## 4. Anatomy of a note

Every note follows the same skeleton, so a reader always knows where to look. Sections that do not apply may be dropped, but their order stays. Notes have no front matter: a note starts with its title.

```markdown
# <Title>

## Summary
Two to four sentences in plain language. What is this, and why should the reader care?

## Prerequisites
Links to notes the reader should know first. State the assumed knowledge.

## Core concepts
Precise definitions, the mechanism, the vocabulary. Build from simple to complex.
Include at least one visual: a diagram, a model or a chart (section 6.2).

## Worked example
A concrete, reproducible example that walks through the idea end to end.

## Trade offs and when to use it
Benefits, costs, alternatives, and the situations where this is the wrong choice.

## Common mistakes
Frequent misunderstandings and failure modes, with the fix for each.

## Practice
Questions or small exercises, with answers or hints kept at the bottom.

## Further reading
Annotated links to primary sources, one sentence on why each is worth reading.
```

Notes on the skeleton:

- **No metadata block.** The directory path tells the area and section, the title is the first line, and open gaps are marked with `TODO:` lines.
- **Visuals sit where they help.** Put the diagram right after the paragraph that introduces the idea, with one sentence saying what to look at. Charts of the numbers in a worked example go directly after its table.
- **Summary first.** A reader who only reads the summary must not come away with a wrong idea. Avoid teasers such as "we will see how this works".
- **Motivation before mechanism.** People remember mechanisms better when they know what they were invented to fix.
- **Practice is part of the note.** Retrieval practice is what turns reading into knowledge. Even three good questions are valuable.

### 4.1 Migrated notes

Notes that were migrated from an older source (an export in `_sources/`) keep their original content and are brought up to the standard over time.

- Sections of the skeleton that the original did not have contain a single line that starts with `TODO:`. Replace it with real content, or delete the section if it does not apply.
- Search for open gaps with `grep -rn "^TODO:" --include="*.md" .`. A note without `TODO:` lines has met the skeleton, but still has to be checked against sources (see [section 8](#8-sources-references-and-verification)).
- Content that was migrated has not been verified against sources.
- A migrated note does not mention where it came from. Do not add "Source material" sections or sentences such as "from the export" or "the original notes say". The only provenance a note carries is its sources list (see [section 8](#8-sources-references-and-verification)).
- The dash and emoji rules of [section 7](#7-writing-style) apply to migrated content as well. Dashes were replaced by hyphens and emoji by words during migration.

## 5. How to explain a concept

Every concept that gets its own section or note is explained with the same five moves. This is the core of the standard.

1. **Definition.** One or two precise sentences. Say what the thing is and what distinguishes it from things that look similar.
2. **Intuition through analogy.** A comparison to something familiar. State where the analogy holds and where it breaks.
3. **Mechanism.** How it actually works, step by step, with the correct terminology.
4. **Concrete example.** A small, realistic case that uses real values and can be reproduced.
5. **Boundaries.** When to use it, when not to, what commonly goes wrong.

### 5.1 Writing good analogies

An analogy is a bridge to a known idea, not a decoration.

- **Map the parts explicitly.** "The index is the book's back-of-book index, the page number is the row location."
- **Choose a familiar source.** Everyday objects and systems work best: libraries, kitchens, traffic, post offices, buildings.
- **Say where it breaks.** Every analogy misleads somewhere. Naming the limit prevents the reader from carrying a false belief forward.
- **Use one analogy per idea.** Stacking three metaphors confuses more than it clarifies.
- **Never let the analogy replace the mechanism.** It opens the door, and the precise explanation must still follow.

### 5.2 Writing good examples

- **Minimal but real.** Small enough to hold in your head, realistic enough to feel like something you would meet in practice.
- **Complete.** The reader can run it or reproduce it without guessing at missing pieces.
- **Show expected output.** Say what happens, including the failure case, so the reader can check their understanding.
- **Progress in difficulty.** Start with the simplest case, then add one complication at a time.
- **Contrast.** Show the wrong way next to the right way, and explain why the first fails.

### 5.3 Depth checklist for a concept

Before a concept is considered explained, you should be able to answer:

- What problem does it solve, and what happened before it existed?
- What are its precise definition and its key properties?
- How does it work internally, at least at one level below the surface?
- What are the alternatives, and how do they differ?
- What does it cost (time, complexity, money, risk)?
- How does it fail, and how would you notice?
- What is the most common misconception about it?

### 5.4 A full example of the standard

The following shows the five moves applied to one concept in this repository.

**Concept: Defense in depth**

*Definition.* Defense in depth is the practice of stacking several independent security controls, so that the failure of one control does not lead to a full compromise.

*Analogy.* Think of a medieval castle. An attacker must cross a moat, climb a wall, get past guards and then break into the keep. Each layer is imperfect, but together they make the attack slow, noisy and expensive. The important detail is that the layers are independent: a drawbridge that fails open should not also disable the wall.

*Example.*

A web application stores customer data in a database. The layers might be:

1. A web application firewall filters obviously malicious requests.
2. The application validates input and uses parameterized queries.
3. The database account used by the application has `SELECT` and `INSERT` on two tables and nothing else.
4. The database is only reachable from the application subnet.
5. Data at rest is encrypted, so a stolen disk is not a stolen dataset.

If an attacker finds a SQL injection in layer 2, layers 3 to 5 still limit what they can read, change or exfiltrate.

*Pitfall.* Treating the layers as a checklist. Five controls that all depend on the same admin password are one control, not five.

Notice the pattern: the definition is precise, the analogy explains the idea in familiar terms, the example is concrete and reproducible, and the pitfall warns about a mistake that real practitioners make.

## 6. Examples, code and diagrams

### 6.1 Code

- Use fenced code blocks and always declare the language (` ```python `, ` ```sql `, ` ```bash `).
- Examples must be **tested**. If you cannot run it, mark it clearly as untested.
- State the **versions** of languages, tools and libraries involved.
- Keep examples **self contained**: imports, setup and sample data included.
- **Comment the why**, not the what. `# retry because the API rate limits at 10 req/s` helps. `# increment i` does not.
- Do not paste large listings. Show the relevant part and link to the full file if needed.
- Output is shown in its own block, labeled as output.

### 6.2 Diagrams, models and visuals

The library should be pleasant to look at and easy to scan. A reader should be able to take in the shape of a topic from its visuals before reading a line of prose.

**What every note has**

- **At least one visual** in "Core concepts" or "Worked example". Notes with several concepts usually need two or three: one overview of the whole topic and one for the central mechanism.
- **An overview for each area and section `README.md`** where it lists more than three notes or sections: a `mindmap` or a `flowchart` of the notes and their order.
- **A chart for any table of numbers** in a worked example, when the numbers are compared or change over time.

**Choosing the form**

| What you show | Use |
| --- | --- |
| A process, a cycle or a decision | `flowchart` (left to right for steps, top to bottom for hierarchies) |
| The parts of a topic or a taxonomy | `mindmap` |
| Messages between actors over time | `sequenceDiagram` |
| Phases, stages or a plan over time | `timeline` |
| Two dimensions, such as effort and impact | `quadrantChart` |
| Numbers compared or trending | `xychart-beta` (bar or line) |
| States and transitions | `stateDiagram-v2` |
| Structure with exact geometry, such as a hexagon model or an annotated figure | hand-drawn SVG in `_assets/<area>/` |

**Rules**

- **Prefer text based diagrams.** Use a fenced `mermaid` block so the diagram is diffable and versioned with the note. Use a fenced `text` block for ASCII diagrams such as trees and data layouts. Mermaid renders on GitHub and in Librarian OS.
- **Keep diagrams small.** Split a diagram when it grows past about 15 nodes. Node labels are short noun phrases or verbs, not sentences.
- **A diagram earns its place** by showing structure, flow over time or relationships that prose cannot. Do not add one for decoration, and do not draw a box around a single idea.
- **Introduce it.** Every visual has a sentence before or after it saying what to look at. The same information is available in the text, so a reader who cannot see the image loses nothing.
- **Only facts from the note.** A visual never adds a claim that the text does not make or source. Invented numbers in a chart follow the same rule as invented numbers in a table: the note says they are examples.
- **Mermaid syntax pitfalls.** Quote labels that contain parentheses, colons or special characters. Avoid em dashes and en dashes in labels, as everywhere. Check that the diagram renders in Librarian OS before finishing.
- **Images and SVG.** Put the file in `_assets/<area>/` named after the note and the subject, keep the editable source in the same directory (`.drawio`, `.excalidraw`, or the `.svg` itself), and add descriptive alt text. A hand-drawn SVG has a `<title>` and `<desc>`, a light background of its own so that it reads in dark themes, and no text smaller than 11 px.
- **Other people's graphics.** Images taken from other people (vendor infographics, slides) are allowed only for personal study. Credit the author in the alt text or caption and prefer redrawing them as Mermaid or SVG when the note is reviewed.

*Example.* Area READMEs use a `mindmap` or a `flowchart` for the map of their notes, the Pod lifecycle note a `stateDiagram-v2` for Pod phases, the API-initiated eviction note a `sequenceDiagram` for the eviction request, and the Horizontal Pod Autoscaler note an `xychart-beta` for the replica counts of a scale-down. Images taken from the Kubernetes documentation are kept in `_assets/` with credit until they are redrawn.

### 6.2.1 Scripts, files and media

- **Scripts** live in a `scripts/` directory next to the topic they belong to (see [section 3.1](#31-layout)). The note shows the relevant part and links to the full script. A script states its language and tool versions in a header comment, is self contained and says how to run it.
- **Downloadable files** (checklists, configuration templates, exported rules, small datasets) go to `_assets/<area>/`. Only files that are safe to publish belong here: no secrets, no real client or personal data, no live malware. Samples, captures and dumps are excluded by `.gitignore` on purpose.
- **Large media** (video, large captures) is linked from a public source instead of being committed. If a large file is essential, discuss it first because it stays in the git history forever.
- **Every asset and script** appears in `ToC.md`.

### 6.3 Tables

Use tables for comparisons across the same dimensions (features, costs, trade offs). Do not use tables for prose that happens to have two columns. Keep cells short.

### 6.4 Data

Use synthetic or public data only. If a dataset is needed, either generate it in the example or link to its public source. Never include personal, confidential or client data.

## 7. Writing style

The writing must sound like a knowledgeable person explaining something to a colleague. It should be plain, direct and specific.

### 7.1 Language

- **Everything is written in English.** This includes notes, headings, comments in code, commit messages, file names and the repository documents (README, GUIDELINE, ToC, AGENTS, CLAUDE). Source material in another language is translated into English when it is migrated.
- Use one variant of English consistently (American spelling is the default here).
- Prefer short sentences. If a sentence needs two readings, split it.
- Use the active voice and concrete subjects. "The scheduler drops the task" beats "the task is dropped".
- Prefer plain words over inflated ones. Write "use", not "utilize". Write "help", not "facilitate".
- Define acronyms on first use in each note, then use the acronym.

### 7.2 Punctuation: hyphens only

Use ordinary hyphens (`-`) and nothing longer. **Do not use em dashes or en dashes** in any file in this repository. They are a recognizable marker of machine generated text and they make prose harder to read.

Instead, rewrite the sentence:

| Instead of | Write |
| --- | --- |
| The cache, which is fast, is not durable | The cache is fast, but it is not durable |
| Two options: fast, or safe | There are two options: fast or safe |
| It works, in theory | It works in theory |

Guidance:

- Use commas, periods, colons and parentheses for pauses and asides.
- Use a hyphen for compound modifiers ("read-only replica", "low-level API") and for number ranges written with digits.
- If a sentence seems to need a dash, it is usually two sentences, or it needs a comma.

### 7.3 Avoiding machine sounding language

Text produced by language models has recognizable habits. Remove them, whether you wrote the text or an assistant did.

- No filler openers: "It is important to note that", "In today's fast-paced world", "Let's dive in", "In conclusion".
- No vague intensifiers or hype: "powerful", "seamless", "robust", "game changing", "cutting edge", "leverage", "delve", "landscape", "tapestry", "crucial" without a reason.
- No empty triads and rhetorical patterns ("fast, scalable, and reliable"), or the "It's not just X, it's Y" construction.
- No summary sentence that only repeats the paragraph.
- No decorative emojis and no exclamation marks.
- No headings that are questions or slogans. Use plain descriptive headings.
- Do not over-format. Bullet lists are for parallel items, not for every thought. Bold is for a term being defined or for a rare warning.
- Be specific. Replace "many companies use this" with a named example, or with nothing.

**Before:**

> It is important to note that database indexes are a powerful tool that can seamlessly boost your query performance, making your applications faster and more scalable!

**After:**

> An index lets the database find rows without reading the whole table. A lookup by customer ID on a table with 5 million rows drops from a full scan to a few page reads. The cost is slower writes, because every insert must update the index.

### 7.4 Formatting conventions

- One top level heading (`#`) per file, matching the title. Then `##` and `###`. Do not skip levels.
- Headings use sentence case: "Query performance", not "Query Performance".
- Use backticks for code, commands, file names, and identifiers. Use *italics* for the first mention of a new term when it is being defined.
- Wrap nothing by hand. Write one paragraph per line, so diffs stay readable.
- End every file with a single newline.
- Numbers: use digits for measurements and counts above nine, write out smaller numbers in prose, and always include units.

## 8. Sources, references and verification

- **Prefer primary sources.** Specifications, official documentation, original papers and source code beat blog posts and videos. Use secondary sources to find primary ones.
- **Sources are listed in "Further reading".** That section is the sources list of the note, and every note keeps it filled in: title, author or organization, year and a link. Structured detail goes into `sources` in the sidecar.
- **No "Source material" sections.** Notes and READMEs do not describe which export, file or earlier note their content came from. The sources list names only real references (papers, standards, official documentation, books).
- **Cite what you use.** Put links in the "Further reading" section, or inline for a specific claim. Include enough to find the source if the link dies: title, author or organization, year.
- **Write in your own words.** Summaries are paraphrased. Quote only when the exact wording matters, keep quotes short, and mark them clearly.
- **Record the date.** For anything that changes over time (versions, prices, limits, recommendations), write the version and the date it was checked in the text, for example "Checked against NIST CSF 2.0, 2026-09".
- **Separate fact, interpretation and opinion.** Mark opinions as opinions ("In my experience", "I prefer this because").
- **Check claims that sound too good.** Reproduce numbers when you can. Look for a second source for anything surprising.

Typical primary sources here are the Kubernetes documentation (`kubernetes.io/docs`), the API reference, Kubernetes Enhancement Proposals (KEPs) in `kubernetes/enhancements`, the source code and release notes in `kubernetes/kubernetes`, the CNCF and its project documentation (etcd, containerd, CRI-O, Helm, Flux, Argo, Prometheus, OpenTelemetry), specifications (OCI image and runtime specifications, CNI, CSI, CRI), papers (Borg, Omega, Raft) and the documentation of vendors for their own products. Blog posts, vendor comparisons and marketing are secondary sources. For security notes see also [section 9.1](#91-security-notes).

## 9. Rules specific to this repository

Kubernetes ships a minor release about every four months. Fields move from alpha to beta to stable, feature gates are removed, APIs are deprecated and whole features (PodSecurityPolicy, in-tree volume plugins, dockershim) disappear. The rules below keep the notes true over time.

- **Upstream first.** Explain the behavior of upstream Kubernetes first. Behavior of a distribution or a managed service (EKS, GKE, AKS, OpenShift) is marked as an example and lives in `ecosystem/` unless it only illustrates an upstream concept.
- **Date and version everything that drifts.** Feature states, defaults, flags, limits and API versions are written with the Kubernetes version and the date they were checked, for example "Checked against the Kubernetes documentation v1.37, 2026-11". Prefer linking to the official page over copying a long table.
- **Mark feature maturity.** Say whether a feature is alpha, beta or stable (GA) and name its feature gate when it has one. Alpha features are off by default and may change without notice.
- **Mark deprecated and removed features.** When a note describes something that is deprecated or removed (PodSecurityPolicy, dockershim, the Flocker volume plugin, Scheduling Policies), say so and name the replacement.
- **Manifests and commands are tested or marked.** A manifest, `kubectl` or `helm` command is either run against a cluster with the versions stated (for example kind with Kubernetes v1.37), or marked as untested. Use `apiVersion` values that are current for the stated version.
- **No real identifiers.** Use placeholders such as `example.com`, `registry.example.com`, `<TOKEN>` and documentation address ranges. Never commit kubeconfig files, tokens, certificates, cloud account IDs or real cluster names.
- **Opinion is marked.** Practices such as GitOps tooling, Helm versus Kustomize or service mesh adoption are debated. Say whether a statement is established, debated or one author's view.
- **One home per concept.** See [section 3.1](#31-layout). A practice note that touches many concepts links to the concept notes instead of repeating them.
- **Drifting topics.** If a note drifts into another area, keep the part that serves its own area and link to the home note of the rest.

### 9.1 Security notes

These rules apply to every note under `security/`, to `services-and-networking/network-policies.md`, to `ecosystem/security-tools/` and to the security sections of other notes.

- **Authorized use only.** Every offensive technique (for example privilege escalation through RBAC verbs or a container escape) must be framed in the context of a lab, a CTF, a penetration test with written permission, or defensive research. State this context explicitly in the note.
- **Defense first.** Each attack note ends with a section on detection and mitigation. Knowing how to break something is only half of the note.
- **No live secrets.** Use placeholders such as `<API_KEY>` and reserved documentation addresses (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`, `example.com`).
- **Reference identifiers.** When a note discusses a known weakness, link the CVE, CWE, OWASP or ATT&CK identifier so the reader can verify the claim.

## 10. Quality checklist

Run this list before marking a note as `reviewed`.

**Content**

- [ ] The summary is correct on its own and written in plain language.
- [ ] Every new term is defined at its first use.
- [ ] The problem the topic solves is stated before the mechanism.
- [ ] There is at least one worked example that can be reproduced.
- [ ] At least one analogy is used, and its limit is stated.
- [ ] Trade offs, alternatives and failure modes are covered.
- [ ] Common mistakes are listed with their fixes.
- [ ] Versions and dates are recorded where relevant.

**Accuracy**

- [ ] Claims are backed by sources, and the sources are linked.
- [ ] Code was run, and the output shown is real.
- [ ] Numbers are reproducible or have a stated origin.

**Style**

- [ ] The note is entirely in English.
- [ ] No em dashes or en dashes anywhere (search for them before committing).
- [ ] No filler phrases, hype words or decorative emojis.
- [ ] Headings are descriptive and in sentence case.
- [ ] Links are relative and work.

**Visuals**

- [ ] The note has at least one diagram, model or chart, and it renders in Librarian OS.
- [ ] Each visual is introduced by a sentence, and shows only facts from the note.
- [ ] Charts match the numbers in the table they illustrate.
- [ ] Images have alt text, and SVG files have a title, a description and their own background.

**Hygiene**

- [ ] No secrets, personal data or confidential material.
- [ ] File and directory names follow the naming rules.
- [ ] The note is linked from its section `README.md`.
- [ ] `ToC.md` reflects every added, moved, renamed or deleted file, and its links resolve.
- [ ] The note has a sidecar (`.md.json`), it follows `INSTRUCTIONS.md` of Librarian OS, and `npm run check` in Librarian OS passes.

A quick way to check the dash rule from the repository root:

```bash
grep -rnP "\x{2013}|\x{2014}" --include="*.md" --include="*.json" --include="*.ts" --include="*.scss" --exclude-dir=node_modules .
```

`npm run check` repeats this test for all text files it reads.

## 11. Maintenance and versioning

- **Small, focused commits.** One note or one logical change per commit. Explain the reason in the message.
- **Commit message format.** A short imperative subject in English, for example `Add note on B-tree indexes`. Add a body when the reason is not obvious.
- **Branching.** Use short lived branches for larger additions. Small edits can go directly to the main branch.
- **Review cycle.** New notes contain `TODO:` lines until the skeleton is filled. After that, check the content against sources and record what was checked in the text.
- **Refresh.** Revisit notes on fast moving topics regularly, update the recorded check date and fix what changed. Delete or rewrite content that is no longer true instead of leaving it with a warning.
- **Refactor structure early.** When a note grows too large, split it. When two notes overlap, merge them or extract the shared part into its own note and link to it.
- **Keep the index honest.** Every new note is added to its section `README.md` and to `ToC.md` in the same commit. Every move, rename or deletion is reflected in both as well.

## 12. Working with AI agents

AI assistants are welcome collaborators. The rules are the same as for a human contributor, and the human author remains responsible for correctness.

- **Read this file first.** `AGENTS.md` and `CLAUDE.md` point here. This GUIDELINE is the only place where conventions are defined.
- **Do not invent facts.** If a detail is uncertain, say so in the note or leave it out. Never fabricate citations, version numbers, benchmarks or quotes.
- **Verify before writing.** Use primary sources for claims, and record them.
- **Follow the style rules strictly.** In particular the ban on em dashes and en dashes, and the ban on filler language (see [section 7](#7-writing-style)).
- **Make it visual.** Add diagrams, models and charts as described in [section 6.2](#62-diagrams-models-and-visuals). A note with only prose and tables is not finished.
- **No "Source material" sections.** Never mention `_sources/` exports in a note. Fill in the sources list instead.
- **Stay in scope.** Do not add content unrelated to the request. Do not restructure the repository without being asked.
- **Prefer editing to creating.** Extend an existing note when the topic already has a home. Create a new file only when the topic deserves its own note.
- **Follow `INSTRUCTIONS.md`.** For every Librarian OS adaptation (configuration, sidecars, entry points, checks) follow `INSTRUCTIONS.md` of Librarian OS, and read it before creating the first note of a session.
- **Keep sidecars in step.** A new or renamed note gets a sidecar (`npm run scaffold`), and relations are added only to ids that exist. Run `npm run check` in Librarian OS before finishing.
- **Update `ToC.md` with every change.** Any file added, moved, renamed or deleted must be reflected in `ToC.md` before the task is finished (see [section 3.4](#34-table-of-contents)).
- **Ask when a decision is unclear.** Structure, scope and naming decisions belong to the author.
- **Never commit secrets or personal data,** and never run destructive git operations without explicit approval.
- **Show your work.** When adding a note, summarize what was written, which sources were used and what remains unverified.
