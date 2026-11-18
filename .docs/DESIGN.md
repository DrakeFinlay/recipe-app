# Finlay Cookbook app design

Date: November 18, 2026

Status: design baseline for implementation. No application features or migration
have been implemented. Product requirements below reflect the user's decisions;
schema, keyboard shortcuts, dependency choices, and other implementation defaults
are proposed engineering choices within those requirements.

## 1. Purpose

Replace the single Google Doc cookbook with a comfortable Windows desktop app
backed by a portable collection of individual Markdown recipes. The app should
make recipes easy to find, maintain a clear distinction between favourites and
recipes to try, and produce a useful PDF cookbook for Google Drive.

Success means the user can open the app from a shortcut, find and read a recipe,
edit its Markdown, and export the favourite cookbook without an internet
connection, login, terminal, or separate development tools.

The working name is **Finlay Cookbook**. Branding can change independently of the
data format.

## 2. Confirmed product decisions

| Area | Decision |
| --- | --- |
| Platform | Windows desktop first, similar in delivery and simplicity to Nakama/Gengo. |
| Connectivity | Everyday use and PDF export work offline. Later explicitly requested imports may use online services. |
| Storage | One unique recipe per Markdown file in an ordinary local library folder. |
| Organization | Alphabetical recipe library, searched and filtered by metadata; no inherited chapter hierarchy. |
| Collections | Favourites and To Try; membership changes only through an explicit user action. |
| Editing | Markdown editor for MVP; a form-based recipe editor is unnecessary. |
| Initial migration | Preserve existing content, including incomplete `(add)` recipes and To add entries. |
| PDF | Favourites only, light background, US Letter, cover, clickable contents, page numbers, each recipe starts on a fresh page. |
| Visual direction | Dark, simple desktop UI with restrained styling inspired by Nakama/Gengo. |
| Deferred | Photo extraction, custom PDF styling/organization, ingredient matching, Drive automation, hosted/mobile access. |

Phone access is a distant possibility. Keep recipe logic separate from the
desktop UI, but do not build a server, web frontend, accounts, or sync for MVP.

## 3. MVP boundary

### Included

1. Select or create a local library, then reopen it on subsequent launches.
2. Perform a reviewed, one-time conversion of the supplied cookbook into recipe
   files, with an inventory explaining incomplete or ambiguous entries.
3. Browse All Recipes, Favourites, and To Try in alphabetical order.
4. Search titles, ingredients, instructions, tips, metadata, and source notes.
5. Filter by recipe type, protein, cuisine/region, tags, and completeness.
6. Read formatted recipes, including local images when available.
7. Create and edit Markdown recipes with preview, save, and unsaved-change handling.
8. Change collection membership explicitly through a visible control.
9. Export the full Favourites collection to a local PDF and open its location for
   manual upload to Google Drive.
10. Handle external file edits, invalid metadata, missing images, and failed writes
    without silently losing content or changing collection membership.

### Deferred

- Automatic extraction from cookbook photos, scans, PDFs, or recipe websites.
- Drag-and-drop collection changes; the explicit collection selector is sufficient
  for MVP.
- Form-based ingredient/instruction editing, serving scaling, unit conversion,
  shopping lists, meal planning, timers, and nutritional calculations.
- Pantry matching or ingredient availability tracking.
- PDF theme editor, custom chapters, manual ordering, and saved export profiles.
- Automatic Drive upload, cloud sync, accounts, hosting, and phone UI.

Adding existing local images manually is allowed in MVP. Recovering an image from
the supplied export is distinct from interpreting a new recipe photo with AI.

## 4. Findings from the supplied cookbook

Source examined: `D:\Drake\Downloads\Finlay Cookbook.md`.

- The export has 31 embedded PNG image definitions using base64 data URLs. Images
  survived, but some appear alongside section headings; not every image should
  automatically become a recipe's finished-dish photo.
- Recipe titles occur at different heading levels. Blank headings, Google Docs
  anchors, formatting escapes, and the old page-numbered contents need special
  treatment during conversion.
- Ingredients, preparation, and tips use inconsistent heading styles. Some
  recipes include ingredient subgroups, yields, personal changes, or source links.
- The text includes fractions, Japanese, Korean, Arabic, and named variants such
  as two versions of Baked Chicken Over Rice. Preserve these distinctions.
- Existing chapters combine meal types, ingredients, and cuisines. They are
  migration clues rather than a required future folder or PDF structure.
- To add contains links and sometimes personal notes. Find new recipes contains
  general source bookmarks. To try primarily contains links and a few text ideas.
- Some entries exist in several places or have only a title. Do not infer that a
  duplicate mention or missing body means the entry should be discarded.

The migration is a separate implementation milestone. This design task does not
modify the downloaded source or copy its contents into the code repository.

## 5. Library and Markdown contract

### Files and ownership

The user chooses a library location outside the source checkout. The app proposes
`Documents\Finlay Cookbook` under the Windows-resolved Documents folder; it does
not assume that Documents is on drive C or that it is cloud-synced.

```text
Finlay Cookbook/
  recipes/
    banana-bread.md
    baked-chicken-over-rice-drake.md
    baked-chicken-over-rice-nena.md
    ...
  assets/
    <recipe-id>/
      finished.png
  references/
    recipe-sources.md
  migration/
    import-report.md
    manifest.json
```

All recipe files share one flat folder. File Explorer can sort by filename; the
app sorts by the displayed recipe title. Category, protein, and cuisine never
determine a recipe's path. Collection changes update metadata and leave the path
unchanged.

Use readable Windows-safe filenames derived from titles, with a short ID suffix
when necessary to avoid collisions, including case-insensitive collisions and
reserved filenames. Preserve the full Unicode title in the file. A title edit
does not automatically rename the file. Stable recipe IDs keep identity separate
from either title or path.

Markdown and local assets are the source of truth. Preferences, editor recovery
drafts, and backups live under `%LOCALAPPDATA%\FinlayCookbook`, outside the code
repository. A searchable in-memory index can be rebuilt from the files; MVP does
not require a recipe database. Copying the library folder preserves the recipes,
images, collections, and references without an application-specific export.

### Example file

This is a synthetic schema example, not a migrated cookbook recipe:

```markdown
---
schema_version: 1
id: "27d00645-c645-42ae-b12e-b666df56a921"
title: "Example chicken and rice"
collection: to-try
completeness: complete
types: [main]
proteins: [chicken]
cuisines: []
tags: [one-pot]
source:
  kind: personal
  reference: "Personal notes"
---

# Example chicken and rice

## Ingredients

- 1 cup rice
- 2 chicken thighs

## Instructions

1. Follow the tested preparation written here.

## Tips

Optional personal notes.

![Finished dish](../assets/27d00645-c645-42ae-b12e-b666df56a921/finished.png)
```

### Metadata semantics

| Field | Meaning and rules |
| --- | --- |
| `schema_version` | Format version, initially `1`; unsupported future versions are reported rather than rewritten. |
| `id` | Stable UUID generated at creation/migration. Duplicates are diagnosed, never silently merged. |
| `title` | Display/search title; preserve native-language names and personal variants. |
| `collection` | Exactly `favourites` or `to-try`. Only this field controls collection membership and export eligibility. |
| `completeness` | `complete` or `placeholder`. Independent from collection: an approved favourite may still need transcription. |
| `types` | Optional list, such as main, side, soup, bread, dessert, drink, sauce, or seasoning. |
| `proteins` | Optional list, such as chicken, beef, pork, seafood, tofu, egg, or legumes; empty is valid. |
| `cuisines` | Optional cuisine/region list. Multiple values and unknown/empty values are valid. |
| `tags` | Optional free-form labels, such as quick, festive, or one-pot. |
| `source` | Optional provenance with a kind and reference; may record a website URL, cookbook/page, or personal origin. |

Use list fields to represent multiple classifications. Suggested labels make
filtering consistent, but the user may add labels; do not hard-code a closed
taxonomy. Metadata is editable as YAML in the Markdown editor. Normalize label
comparison for filtering without rewriting the user's spelling on disk.

The front-matter title is authoritative for lists and the PDF contents. A missing
or differing first H1 produces a helpful diagnostic; do not rewrite recipe text
silently or print the title twice. The app's new-recipe template makes them match.

Recipe bodies remain flexible Markdown. Prefer Ingredients, Instructions, and
Tips headings for new recipes, but accept imported Preparation headings, grouped
ingredients, source links, and other useful notes. Preserve quantities as text;
do not require a structured ingredient parser for MVP.

### Invalid and externally edited files

- A `.md` file without valid required metadata stays accessible under All Recipes
  with a Needs attention marker. It is not assumed to be a favourite or silently
  assigned to a collection.
- Invalid YAML can be saved as a draft/recovery copy; replacing a valid library
  file requires fixing the errors. Unsupported optional fields are preserved.
- Watch the library for changes and offer an explicit Refresh command as well.
  Refreshing updates clean previews and search results.
- If a file changes externally during editing, offer reload, compare, or Save a
  copy. Never overwrite the changed original silently.
- Save through a temporary file and atomic replacement where supported; leave
  the original intact when saving fails. Retain a recoverable previous version
  before replacing existing recipe content.
- Resolve image references locally relative to the recipe file. New images are
  copied into the library's assets folder. Never auto-fetch remote images for
  reading or exporting; report them and allow local attachment instead.
- Render ordinary Markdown as content. Do not execute scripts or embedded HTML.

## 6. Collections and initial migration

Collection membership is an explicit piece of recipe data. Cooking a recipe,
editing it, completing its placeholder, or importing a photo must not promote it
to Favourites. MVP uses a Favourites / To Try selector with an immediate Undo
action. A later drag-and-drop interaction must call the same explicit operation.

New manually created recipes default to To Try. The user can deliberately select
Favourites during creation. An incomplete recipe can belong to either collection.

### Conversion rules

1. Read the original export without modifying it. Convert into a staging library
   and generate a reviewable inventory before replacing any existing library data.
2. Identify recipe boundaries from the contents, headings, and body structure
   together. Ignore blank/image-only headings as recipe boundaries. Resolve
   ambiguous cases in the report rather than guessing from heading depth alone.
3. Convert existing cookbook recipes to Favourites. Keep `(add)` in titles and
   preserve their existing bodies, including empty bodies; mark incomplete entries
   as placeholders. Do not invent ingredients or instructions.
4. Convert individual To add items to favourite placeholders, preserving their
   URLs and attached personal notes exactly. When no title exists, use a readable
   provisional title derived from the URL or idea and flag it for review.
5. Convert individual To try ideas/links to To Try placeholders. Preserve related
   links together when the source clearly describes one recipe with alternatives;
   ambiguous grouping goes in the report.
6. Preserve Find new recipes as `references/recipe-sources.md`, accessible through
   Open library folder. General source websites are not individual recipes and
   do not appear in the recipe search set or cookbook PDF.
7. Preserve recipe wording, units, notes, native-language text, and named variants.
   Structural changes are limited to splitting files, adding front matter, setting
   the recipe's title heading, and rewriting image references to extracted assets.
   Record any necessary additional cleanup rather than silently rewriting content.
8. Extract all 31 image definitions into local files. Associate recipe images only
   where their placement makes ownership clear; retain ambiguous/decorative images
   in a migration asset area with report entries. An absent photo never blocks a
   recipe from being imported or read.
9. Use clear source headings as suggestions for type/protein/cuisine metadata,
   without reconstructing their hierarchy. Leave uncertain values empty and
   reviewable; never invent cuisine from a title alone.
10. Flag possible duplicates across sections with their source locations. Keep
    separate entries until deliberately resolved; do not merge personal variants
    or promote a To Try entry because a favourite resembles it.
11. Record source locations, content fingerprints, output IDs/paths, original
    section, collection decisions, image associations, and unresolved issues in
    the migration manifest/report. Rerunning the same import must not duplicate
    entries or overwrite subsequent edits.

Review reconciles every source recipe/placeholder, reference section, and image
to an output file or explicit unresolved item. No numerical recipe total is
promised until the actual migration inventory is complete.

## 7. Desktop user experience

### Main window

Use a restrained three-area desktop layout:

```text
+---------------+----------------------+-----------------------------------+
| Finlay        | Search recipes       | Selected recipe title             |
| Cookbook      | Filters / Sort A-Z   | Collection / metadata             |
|               |                      |                                   |
| All Recipes   | Alphabetical results | Read | Edit                       |
| Favourites    | Title                |                                   |
| To Try        | Metadata / status    | Formatted recipe, or Markdown     |
|               | ...                  | editor with optional preview      |
| New Recipe    |                      |                                   |
| Export PDF    |                      |                                   |
| Settings      |                      |                                   |
+---------------+----------------------+-----------------------------------+
```

The sketch describes interaction and hierarchy, not final dimensions. The
editor can expand into the results area on smaller windows. Reading is the
default; editing is a deliberate mode with a clear unsaved indicator.

### Find and read

- Start in Favourites, restore the last selected recipe when possible, and show
  a helpful empty state for a new library. Collection counts remain visible.
- Sort titles alphabetically by default; offer reverse alphabetical order.
- Search the selected collection across title, body, metadata, and source notes.
  Terms use case-insensitive AND matching; punctuation/fraction content remains
  intact. Accent-insensitive matching is desirable for Latin text. No online
  semantic search, stemming, or translation is required.
- Filters combine with search. Different fields use AND; selected values within
  one field use OR. Show active filters, result counts, and Clear filters.
- Support an Unknown/Not set filter so unclassified recipes are findable.
- Results show the title, collection, selected metadata, and a placeholder or
  attention badge. Use a compact text list; images are optional recipe content
  rather than a prerequisite for a usable gallery.
- The reader presents selectable text, ingredients, numbered steps, tips, and
  available images with clear spacing. Source links open in the default browser
  only when clicked; they are optional conveniences while offline.

### Create, edit, and change collection

1. Select New Recipe and supply a title and initial collection; generate a file
   draft containing the metadata and section template.
2. Edit Markdown directly, including metadata. Offer a preview toggle or split
   preview using the same parsed content as reading mode.
3. Save with a button or Ctrl+S. Validate metadata and display actionable errors
   beside the editor rather than a crash dialog.
4. Prompt Save / Discard / Cancel before navigation or closing would lose changes.
   Recovery drafts support an interrupted editing session.
5. Use the collection selector to move a saved recipe explicitly. If there are
   unsaved changes, resolve them first rather than altering a stale file copy.
6. Provide Open file and Open library folder for external editing and backup.

Use Ctrl+N for a new recipe, Ctrl+F to focus library search, and Escape to leave a
temporary overlay. Keep normal text selection, tab navigation, visible focus,
high-DPI scaling, and adjustable reading/editor font size. Destructive deletion
is not needed for MVP; recipes can be removed through the library folder and
the refreshed app must reflect this safely.

### Styling

Use a dark neutral background, slightly elevated panels, readable off-white text,
muted secondary labels, and one restrained accent color. Follow the spacing and
sidebar simplicity of Nakama/Gengo, with minimal borders and no decorative
dashboard. Distinguish selected, hovered, focused, and disabled controls. Errors
and collection states need text labels in addition to color.

Final color tokens and spacing should be checked against those apps during UI
implementation. The document does not claim that a visual prototype exists.

### Settings

MVP settings contain the library path, Open library folder, Refresh library,
reading/editor font sizes, and last-used export directory. Choosing a different
library opens that folder; it does not move, merge, or delete the previous one.
Handle a moved/unavailable library with Locate library / Create library actions
without resetting or silently replacing saved data.

## 8. PDF cookbook

### Export flow

1. Select Export PDF from anywhere in the app.
2. Show the fixed scope, **Favourites only**, recipe count, placeholder count,
   missing-image notices, and chosen output path. Search and active list filters
   never narrow this full-cookbook export.
3. Resolve unsaved edits before taking an immutable snapshot of saved favourites
   and their local assets. Validate the snapshot; malformed collection metadata
   or unresolved recipe identity blocks export with a repair list rather than
   silently producing an incomplete or mixed cookbook.
4. Generate into a temporary file, then publish the completed PDF to the chosen
   path. Confirm overwriting an existing PDF and leave it intact on failure.
5. Offer Open PDF and Open containing folder after success. The user manually
   uploads the resulting file to Google Drive.

No internet connection, Drive authorization, or print driver is required for
generation. Export errors remain in the app and preserve the library.

### Initial layout

- US Letter portrait, white background, dark text, comfortable print margins.
- Cover titled Finlay Cookbook; a simple export date is acceptable.
- Alphabetical clickable table of contents with correct recipe start page numbers.
- Alphabetical recipes without inherited chapters. Each starts on a new page;
  long recipes may continue across several pages. Short recipes do not share a
  page with the next recipe.
- Consistent title, headings, ingredient bullets, numbered instructions, tips,
  and local images. Keep headings with following text and avoid clipped content.
- Page numbers after the cover. Contents links and displayed page labels must
  agree, including when the contents itself spans multiple pages.
- Preserve source notes and personal changes. Hide YAML, file paths, diagnostic
  metadata, and the app's navigation from the printed recipe.
- Include favourite placeholders with their existing title/content and a small
  Incomplete recipe notice. Warn before export; do not silently omit them or
  manufacture missing steps. To Try entries remain excluded regardless of
  completeness or tags.
- Missing optional photos produce a notice, not a failed recipe or blank image
  box. Unsupported characters and lost text are export defects, not acceptable
  fallback behavior.

The PDF has its own styling, independent of the dark app and the Markdown files.
Later customization can group recipes by metadata, define chapter order, adjust
fonts/margins/colors, and manage image size/visibility. Those choices belong to
export profiles, not filesystem paths or rewritten recipe content.

## 9. Coding stack and architecture

### Selected direction

| Component | Choice | Reason |
| --- | --- | --- |
| Application language/runtime | C# on .NET 10 | Matches the existing desktop projects and keeps domain, files, and export in one language. |
| Desktop UI | WPF, XAML, MVVM-style view models | Fits the Windows-only target and the established app interaction model. |
| Recipe parsing | Markdig | Parse Markdown into a syntax tree shared by reader and export adapters. |
| Metadata parsing | YamlDotNet | Read and validate explicit YAML front matter rather than ad hoc regular expressions. |
| PDF generation | QuestPDF, subject to an early real-content rendering check | Native .NET document layout with separate cookbook styles and local generation. |
| Preferences/recovery | .NET JSON and local files | Small app-owned state without introducing a server or primary recipe database. |
| Search | Rebuildable in-memory index | Sufficient starting point for a personal library; add SQLite only if measured scale requires it. |
| Packaging | Self-contained Windows x64 folder and a shortcut | Include the runtime so ordinary use needs no SDK or separately installed .NET. |
| Development tooling | .NET SDK, PowerShell, Git | Consistent build/check/publish workflow with the neighbouring projects. |

WPF's Windows scope and XAML/data-binding model are documented by
[Microsoft](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/overview).
Self-contained publishing includes the required runtime; see
[.NET deployment](https://learn.microsoft.com/en-us/dotnet/core/deploying/).

[Markdig](https://github.com/xoofx/markdig) supplies the Markdown syntax tree, and
[YamlDotNet](https://github.com/aaubry/YamlDotNet) supplies YAML parsing. Using them
does not itself provide the WPF preview or preserve arbitrary YAML formatting:
those are adapter and save-behavior responsibilities to verify explicitly.

[QuestPDF's font guidance](https://www.questpdf.com/api-reference/text/font-management.html)
supports explicit font registration/fallback; package appropriate licensed fonts
for Latin, fractions, Japanese, Korean, and Arabic rather than relying on a font
download at first use. Its proposed use for this personal app is based on the
vendor's [Community License](https://www.questpdf.com/license/community.html),
which lists individuals as eligible. Record the actual dependency versions and
license texts when packages are selected; reassess the license if distribution
or ownership changes. Do not describe QuestPDF as MIT-licensed.

Before building the complete PDF workflow, verify Arabic shaping/direction,
Japanese/Korean glyphs, clickable contents with page references, local images,
and long recipe pagination using representative content. The PDF engine remains
replaceable if that check fails; do not compromise preserved recipe text to fit
an engine. No package versions or successful rendering are claimed at design time.

### Boundaries

```text
WPF views / view models
          |
Recipe service / search / collection commands / export orchestration
          |
Recipe model + metadata validation + parsed Markdown
          |
Filesystem library | migration | WPF reader adapter | PDF renderer
```

Use a small solution with Core, Infrastructure, and App projects plus focused
checks. Core has no WPF dependency. The filesystem repository owns enumeration,
atomic writes, change detection, and collision handling. Collection operations
share the same save path as editing. Preview and PDF consume the same parsed
recipe model but have independent layout code.

The Markdown support contract initially covers headings, paragraphs, emphasis,
links, local images, ordered/unordered and nested lists, block quotes, and simple
tables. Inventory any source constructs outside that set before migration;
preserve their text and add support or report a rendering gap. Unsupported raw
HTML never becomes executable UI.

A later hosted version can reuse models and parsing, but WPF screens would need
an alternative frontend. No promise is made that desktop UI code will run on a
phone unchanged.

## 10. Implementation milestones and acceptance criteria

### M0: Library contract, migration inventory, and PDF feasibility

- Implement schema validation, recipe identity, and a synthetic library fixture.
- Inventory the source export and produce a staged migration/report without
  modifying the original or losing placeholders, variants, bookmarks, or images.
- Exercise PDF output with non-Latin text, multi-page recipes, nested ingredients,
  images, and a contents section long enough to span multiple pages.
- Accept when source-to-output reconciliation is reviewable and the selected
  PDF renderer preserves the required content.

### M1: Usable offline reader

- Open a selected library and display alphabetical collections, recipes, metadata,
  local images, full-text results, and combined filters.
- Accept when the packaged app launches with networking unavailable and can find
  and read complete recipes and placeholders without optional services.

### M2: Markdown authoring and explicit organization

- Add creation, editing, preview, safe saves, recovery, refresh, and the collection
  selector with Undo.
- Accept when recipes can be edited externally and in the app without silent
  overwrite; moves persist across restarts and never happen automatically.
- Prove missing/invalid metadata cannot accidentally turn a recipe into a favourite.

### M3: Export and MVP release

- Complete favourites-only export, diagnostics, overwrite handling, navigation to
  the output, self-contained packaging, and the launcher shortcut.
- Accept when a real migrated cookbook has the correct scope, alphabetical
  contents, working links/page numbers, a fresh page for each recipe, readable
  fonts, and no clipped images or missing text. Favourite placeholders are
  preserved and clearly identified; no To Try recipe appears.
- Check reading, editing, search, metadata changes, and PDF export while offline.
- Deliver user instructions, build/check/publish guidance, dependency notices, and
  migration results. Update this document to distinguish completed behavior from
  remaining plans.

Focused automated checks should cover collection isolation, invalid metadata,
save conflicts, filename collisions, migration idempotency, search/filter rules,
and PDF scope/page references. Visually inspect desktop layout and representative
PDF pages in addition to checking generated files programmatically. Use synthetic
fixtures in Git; personal cookbook validation stays local.

## 11. Later milestones

1. **Cookbook photo extraction:** select one or more images, optionally call an
   online service, generate a local draft, review it beside the source, correct
   quantities/steps, and save explicitly. New extractions default to To Try unless
   the user chooses Favourites. Attach provenance and retain the original scan.
   Provider setup, supported languages, costs, and batch behavior are future
   decisions; lack of a provider never blocks ordinary app use.
2. **PDF customization:** saved themes and export profiles, custom chapter
   groupings/order, typography, margins, image treatment, and preview. Keep the
   Favourites/To Try separation enforced for full favourite-cookbook exports.
3. **Ingredients on hand:** match against the local recipe library, distinguish
   available/missing ingredients, let the user define pantry staples, and state
   substitution/quantity limitations. Begin with user-reviewable matching rather
   than a promise that a recipe is guaranteed cookable.
4. **Google Drive automation:** explicit opt-in setup, upload destination, and
   clear replacement/version behavior while retaining local export.
5. **Hosted/mobile access:** revisit hosting, authentication, backup, concurrent
   editing, and sync only when that work becomes a real priority.

No additional product decisions block the MVP design. Remaining implementation
choices should follow this baseline and be recorded when resolved.
