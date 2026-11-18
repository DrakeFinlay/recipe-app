# Finlay Cookbook app

A planned Windows desktop app for maintaining a personal cookbook as individual
Markdown files. Browse, search, filter, edit recipes, and export favourites to a
PDF for manual upload to Google Drive.

**Status: design established; implementation has not started.** There is no
executable, build command, or cookbook migration in this repository yet.

Read the [design document](.docs/DESIGN.md) for the MVP, user experience, recipe
format, migration rules, coding stack, acceptance criteria, and future milestones.
It records the decisions discussed on November 18, 2026, alongside implementation
defaults that can be revised as the app takes shape.

The selected application stack is C# / .NET 10 with WPF and XAML. Recipes and their
metadata remain ordinary local Markdown files. The planned self-contained Windows
package will run offline without a development SDK or cloud account.

MVP includes:

- Migration of the existing cookbook, including incomplete entries.
- Alphabetical browsing, full-text search, and metadata filters.
- Markdown editing and a readable preview.
- Explicit movement between Favourites and To Try.
- Manual times-cooked tracking with +1 and an editable total.
- A favourites-only PDF with a cover, linked contents, page numbers, and a fresh
  page for each recipe.

Later work includes photo-to-recipe extraction, PDF styling and organization
controls, ingredient matching, Google Drive automation, and possible mobile access.

Personal recipe data will live in a user-selected library folder outside this
source repository. No source cookbook content has been copied into Git.
