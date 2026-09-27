---
kind: playbook
domain: experience
topics: [form, autosave, explicit-save, inline-edit, wizard, editing, experience, interaction, patterns]
authority: guidance
---
# Editing Pattern Selection

## Explicit save form
Use when users make a deliberate group of changes, validation spans fields, or committing changes has meaningful side effects. Clearly communicate dirty, saving, success and error states.

## Autosave
Use for document-like editing, settings or builders where frequent incremental persistence protects work and explicit commit adds friction. Define debounce, save status, offline behavior, conflicts and recovery.

## Inline editing
Use for small contextual values when preserving the surrounding view matters. Make editability discoverable and provide keyboard/focus behavior.

## Wizard
Use when steps depend on earlier decisions, the full task is cognitively heavy, or progress through required setup benefits from staged validation. Preserve progress and allow appropriate backtracking.

## Bulk editor
Use when the same field/action applies across many selected records and users can preview the scope and consequences.

Select based on task complexity, reversibility, validation dependencies, frequency, data loss risk and user expectation.
