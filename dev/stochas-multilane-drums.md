✅ CODENAME: STOCHAS-DRUMMODE IMPLEMENTATION BRIEF

Objective

Add a new “Drum Mode” to Stochas using the existing codebase.
This mode displays multiple lanes (fixed drum notes) vertically in a single grid, reusing existing step editing features (probability, velocity, nudging, repeats, microtiming, etc).

The goal is to minimally modify the code, by reusing the existing grid component and its step-editing logic.
The task is analysis first, then generate a recommended implementation plan.

✅ SPECIFICATION FOR DRUM MODE
High-Level Behaviour

Introduce a toggle (“Drum Mode”) in the UI.

When Drum Mode is ON:

The grid should display fixed drum lanes instead of pitched notes.

Each lane = one “NoteRow” (kick, snare, hats, etc).

Rows are shown simultaneously (8–12 rows).

Each row has its own sequence of steps (on/off, prob, velo, nudge…).

Step grid editing behaviour remains unchanged.

Existing per-step attributes remain available (prob, velo, nudge, retrig).

Length of each row can be independent or shared with the underlying Layer (depending on what is easiest in the codebase).

No pitch selector or “add note row” UI appears in Drum Mode.

When Drum Mode is OFF:
→ Stochas behaves normally.

User Interface Expectations

Each row must show:

Step on/off

Step divisions

Probability/velocity indicators in existing graphical style

Step nudging/microtiming indicators

Lane labels shown in place of pitch names (e.g., Kick, Snare, CHat…)

Constraints

No breaking of existing Stochas behaviour.

Old projects must load normally when Drum Mode is OFF.

Prefer composition over modification (add a mode rather than rewriting core classes).

✅ CODING TASKS FOR CODEX
Task 1 — Map the Real Data Model

Search the repo for the following classes and read their declarations:

Pattern

Layer

NoteRow or equivalent (class representing rows inside a layer)

Step

The grid rendering component (PatternEditorComponent, SequenceEditor, GridComponent, etc.)

The scheduling/transport playback class

Output:
A short structured summary of:

How Layers, NoteRows, and Steps are actually represented in code

Which class owns the array of steps

How many levels of hierarchy exist

Which class draws the grid

Which class handles click/drag events

Which class handles mode switching (prob/velo/nudge/etc.)

This must be precise and based on the actual source.

Task 2 — Identify the Minimum Intrusive Hook Points

Codex should determine:

Where the list of NoteRows is built (currently pitch-based).

Where grid rows are generated.

Where per-row step arrays are rendered.

Where per-step attributes are modified.

Where to safely inject Drum Mode switch.

Output:
A list of hook points with file names + line numbers + recommended extension method.

Task 3 — Drum Mode Architecture Decision

Codex must determine:

Whether NoteRows can be repurposed directly as lanes without modifying the underlying data structure.

Whether Layers or NoteRows hold per-step data.

Whether per-NoteRow lengths exist or must be added.

The easiest way to lock the number of NoteRows in Drum Mode.

Output:
A recommended solution (expected to be: “repurpose NoteRows as lanes”), with explanation referencing actual code.

Task 4 — Produce a Concrete Implementation Plan

Codex should produce a step-by-step implementation plan:

Add Drum Mode flag to UI state.

Modify grid row generation

In Drum Mode: generate fixed NoteRows instead of pitch-based NoteRows.

Modify rendering

Replace pitch names with lane names.

Grey out add/remove note row UI elements.

Modify data initialization

When entering Drum Mode: create fixed NoteRow objects with pre-assigned MIDI notes.

Hide pitch-selection UI.

Per-row length logic

Bind each row’s length either to the underlying Layer length (easier) or extend NoteRow to hold per-lane lengths.

Serialization plan

Ensure Drum Mode patterns remain backward compatible (e.g., wrap in a single layer with a special flag).

Output:
The full step-by-step plan as a well-structured Markdown document.

Task 5 — Provide a Minimal Patch Draft

Codex should produce:

Suggested code insertions for:

Drum Mode toggle

Drum Mode row binding

Rendering modifications

No huge diffs required — only minimal example snippets showing how to hook into the existing classes.

Final Output Expected from Codex

A single combined deliverable containing:

Actual data model summary

Hook point list (files + line numbers)

Architecture choice justification

Complete implementation plan

Minimal code patch scaffolding

Potential pitfalls (e.g., serialization, UI scaling, compatibility)

This is the “blueprint” you will implement.

