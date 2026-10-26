# Documentation Index

Use this directory as the project knowledge base.

## Sections

- `product/`: what the system does and why.
- `architecture/`: how the system is built and constrained.
- `quality/`: change lifecycle, testing, review, version control, performance, observability, technical debt, and done criteria.
- `security/`: security principles, access control, hardening controls, and threat model.
- `generated/`: generated or derived references.

## Maintenance Rule

When code changes behavior, update the relevant document in the same pull request.

For interface work, answer the screen brief in `architecture/cognitive-architecture.md`
first, then start with `architecture/wpf-mvvm.md`, open only the relevant interface-pattern
documents, and finish with `quality/ui-checklist.md`.

For data work, start with `architecture/database.md`; for sensitive operations, answer the
brief in `architecture/dotnet-architecture.md` first.
