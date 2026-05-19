# Stochas Drum Mode Blueprint

## Data Model
- `SequenceLayer` owns patterns; `Pattern::mRows[SEQ_MAX_ROWS]` -> `Row::mSteps[SEQ_MAX_STEPS]` (`SequenceData.h:85-139`). Each `Cell` holds `prob/velo/length/offset`.
- Patterns per layer: `mPats[SEQ_MAX_PATTERNS]` with shared `mNumRows/mNumSteps` (`SequenceData.cpp:456-485`).
- Per-row note metadata: `NoteSet` (standard/custom) with MIDI note + label; `setNote/getNote` and `setNoteName/getNoteName` (`SequenceData.h:299-336`).
- UI grid draw: `StepCpt::paint` (`StepPanel.cpp:20-110`); overlays in `StepPanel::paintOverChildren` (`StepPanel.cpp:540-612`).
- UI events: `StepPanel::mouseDown/Drag/Up` apply edits and swap buffers (`StepPanel.cpp:587-977`); visibility via `StepCpt::isStepInRange` (`StepPanel.cpp:114-139`).
- Row labels: `NotePanel::refreshAll` populates names/visibility from `SequenceLayer` notes (`NotePanel.cpp:84-140`); dragging labels updates row MIDI note (`NotePanel.cpp:170-224`).
- Mode switching: `EditorState::setEditMode` + UI toggle dispatch (`EditorState.cpp:47-50`, `PluginEditor.cpp:744-781`).
- Scheduling: `StochaEngine` reads layer rows/steps and schedules MIDI (`StochaEngine.h:26-160`); playback driven via `SeqAudioProcessor`.

## Hook Points (file:line – extension suggestion)
- `SequenceLayer::clear` (`SequenceData.cpp:456-485`): inject drum-lane init (fixed notes/names) when flag set.
- Note metadata APIs (`SequenceData.h:299-336`): add helper `applyDrumLayout()` to set fixed notes/labels and lock row count.
- Layout sizing `PluginEditor::resized` (`PluginEditor.cpp:348-422`): clamp step/note holder heights/scroll ranges when drum mode fixes row count.
- Visibility gate `StepCpt::isStepInRange` (`StepPanel.cpp:114-139`): ensure it follows drum row count.
- Label render `NotePanel::refreshAll` (`NotePanel.cpp:84-140`): swap pitch names for drum lane labels; disable pitch editing when drum mode on.
- Options UI `OptionsPanel::cptValueChange`/`refresh` (`OptionsPanel.cpp:440-520`): hide/disable num-rows control under drum mode.
- Mode toggle wiring `PluginEditor` constructor + `cptValueChange` (`PluginEditor.cpp:107-172`, `744-781`): add drum-mode toggle and state propagation.
- Persistence: pattern/layer store/load (`Persist.cpp:250-337`) to add optional drum-mode flag (default off).

## Architecture Choice
- Reuse existing note rows as drum lanes. Steps already per-row/per-pattern; no new containers needed.
- Per-row data lives in `Pattern::Row::mSteps`; layer shares `mNumSteps` (no per-row length), so bind drum lanes to layer length.
- Lock lane count by setting `mNumRows` and preventing UI changes; keep using custom note set for drum note assignments.

## Implementation Plan
- **State**: add `EditorState::mDrumMode` with getter/setter; serialize in config/preset defaulting false.
- **UI toggle**: add “Drum Mode” toggle to toolbar near View selector; on change set state, call drum layout apply, `updateUI()`, repaint.
- **Drum layout**: add helper (e.g., `SequenceLayer::applyDrumLayout(const DrumLayout&)`) that sets `setNoteSource(true)`, writes fixed MIDI notes/labels, sets `setMaxRows(fixedCount)`. Provide static array of common lanes (Kick 36, Snare 38, CH 42, OH 46, Tom1 48, Tom2 50, Rim 37, Clap 39, Perc 60, Ride 51, Crash 49, Cowbell 56).
- **Grid rows**: in drum mode, ignore add/remove rows; ensure `PluginEditor::resized` and scrollbars use fixed row count; `StepCpt::isStepInRange` relies on `getMaxRows()` (kept in sync by layout).
- **Rendering**: `NotePanel::refreshAll` shows lane labels, hides inline pitch edit/drag; keep play buttons if desired. Skip chord overlays in `StepPanel::paintOverChildren` when drum mode.
- **Editing**: reuse existing step editing; only guard chord handler/scale interactions when drum mode. No engine changes required.
- **Persistence**: store/load drum-mode flag per layer (e.g., `drumMode="1"`). Default missing -> false to keep backward compatibility. Pattern data unchanged.
- **Toggling off**: on disabling drum mode, restore previous note set (cache last standard/custom or rebuild from scale) and allow row count control again.

## Minimal Patch Scaffold
- `EditorState.h/.cpp`: add `bool mDrumMode{false}; bool getDrumMode() const; void setDrumMode(bool);` and include in config read/write.
- `PluginEditor`:
  - Add toggle control `mDrumToggle` with ID `SEQCTL_DRUM_MODE`; position near `mEditToggle`.
  - In `cptValueChange`, handle `SEQCTL_DRUM_MODE`: set state, call `applyDrumLayoutIfNeeded`, `updateUI()`, `repaint()`.
- `SequenceLayer`:
  - Add struct `DrumLayout { int count; int8_t notes[SEQ_MAX_ROWS]; const char* names[SEQ_MAX_ROWS]; };`
  - Add `void applyDrumLayout(const DrumLayout&);` writing notes/names and `setMaxRows`.
- `NotePanel::refreshAll`: if `getDrumMode()`, force label visibility to drum names, disable inline edit/drag (skip creating `InlineEditor`, ignore mouse drag).
- `OptionsPanel`: when `getDrumMode()`, hide/disable `mNumRows` control.
- `Persist`: add drum flag to layer XML (`<layer drumMode="1">`) and read it back; default to false when absent.

## Pitfalls
- Backward compatibility: default flag false; avoid changing pattern structure so old presets load identically.
- UI layout: when row count shrinks, ensure `invisibleHeight` math in `StepPanel::resized` stays non-negative and scroll ranges clamp to new row count.
- Interaction: chord handler and scale-based pitch edits should be disabled in drum mode to prevent unintended pitch writes.
- Thread safety: all drum-mode data mutations must go through UI buffer then `SeqDataBuffer::swap()`; avoid touching audio buffer directly.
- Layout intent: drum mode is primarily a presentation/state change. Stack fixed drum lanes vertically in a single layer view instead of paging through pitches. Keep the single-layer row/step model; only use multiple layers if you explicitly need different pattern lengths per drum group.
