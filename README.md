# ComfyUI-AppBuilder

Turn a ComfyUI workflow into a focused, easy-to-use web application without enabling Node 2.0.

This fork expands AppBuilder for day-to-day generation work: full-quality result previews, reproducible local history, prompt tools, workflow text feedback, execution profiling, improved LoRA controls, and a more capable Configure Panel.

**[[📃 中文版](./README_zh.md)]**

## Preview

| Desktop UI | Mobile UI |
|---|---|
| <video src="https://github.com/user-attachments/assets/0abd3b8a-5fc7-40fc-a71f-aec98a1cdea9"> | <video src="https://github.com/user-attachments/assets/384ed02e-be62-4614-a96d-2abd25da67a2"> |

## Installation

Clone the repository into the ComfyUI custom nodes directory and restart ComfyUI:

```bash
cd ComfyUI/custom_nodes
git clone <fork-repository-url> ComfyUI-AppBuilder
```

Existing installations can be updated with `git pull` from the repository directory.

## Basic usage

- Add an **AppBuilder** or **AppBuilder (Advanced)** node to a workflow.
- Open **Configure Panel** and add the workflow parameters that should be visible in AppView.
- Connect the generated outputs through **Parameters Unpacker** to the corresponding workflow inputs.
- Open AppView from the AppBuilder node.
- To expose saved workflows in the standalone App Manager, save them under the `app/` subfolder, for example `app/SD1.5_txt2img`.

## Changes in this fork

### Full-quality preview and batch navigation

The central preview now switches from the low-resolution live preview to the original output as soon as generation finishes. A late live-preview frame cannot overwrite an already available final result.

For a batch:

- all completed images or videos are displayed in a result grid;
- selecting a tile expands it to the full preview area;
- the left and right arrow keys move between focused results;
- `Esc` returns from a focused result to the grid;
- double-clicking a focused result opens the lightbox.

This removes the need to find every completed result in the small history strip before inspecting it at full quality.

### Persistent gallery and generation parameter snapshots

The bottom gallery is retained in browser storage instead of being discarded after a server restart or an execution error. When a generation is queued, AppView records the values of the controls exposed by the current app and links that snapshot to the resulting media.

In the lightbox, **Parameters** shows the stored control values and provides **Restore parameters**. Restoration updates the AppView controls and the corresponding workflow widgets. A restored seed is deliberately changed to **fixed** mode so that the next run does not randomize it again.

Snapshots are limited to the latest 200 generations. Clearing the gallery also clears its linked parameter snapshots.

> Parameter snapshots are stored in the current browser's `localStorage`; they are not embedded into image metadata. Moving a file to another browser or machine does not move its AppView snapshot.

### Improved lightbox

The lightbox now provides:

- filename;
- original resolution;
- aspect ratio;
- file size when it can be obtained from the media URL;
- previous/next navigation with buttons or arrow keys;
- parameter inspection and restoration;
- download, wheel/pinch zoom, drag while zoomed, and `Esc` to close.

Images and videos are both supported.

### Prompt history and safer text editing

Every multiline STRING control has a prompt-history button. A prompt is recorded when generation starts, not on every keystroke.

The history window supports:

- restoring a previous prompt with one click;
- deleting individual entries;
- marking entries as favorites;
- filtering to favorites only;
- a stable, scrollable layout for long histories;
- `Esc` to close.

Up to 100 regular prompts are retained per multiline control and app. Favorites do not count toward that limit and are not removed when regular entries are trimmed. Prompt history is stored locally in the browser.

The multiline control itself is taller, can be resized vertically, and places the full editor button before the history button. In the editor modal:

- `Esc` closes without saving;
- `Cmd/Ctrl + Enter` saves and closes;
- the modal consumes the shortcut, so it cannot accidentally start a generation using the old prompt.

### Generation shortcut and configurable control hotkeys

`Cmd + Enter` on macOS and `Ctrl + Enter` on other platforms start generation in both embedded and standalone AppView, matching the familiar ComfyUI shortcut.

Configure Panel also supports per-control hotkeys for:

- **BOOLEAN** — toggle the value;
- **INT** — increment by the configured step while respecting minimum and maximum values.

Click the hotkey field and press the desired combination. `Backspace` or `Delete` clears it. The global `Cmd/Ctrl + Enter` shortcut is reserved for Run.

### Generation time and per-node profiling

Elapsed generation time is shown next to the status indicator. Hovering it displays the five nodes with the highest measured execution time. Clicking it opens the complete execution list in call order with per-node durations.

Timings are measured from ComfyUI execution events. Cached nodes are not reported, so the node durations are diagnostic estimates rather than a low-level GPU profiler.

### Workflow text feedback

The new **AppBuilder Text Output** node lets AppView display text produced inside a workflow, such as an LLM-expanded prompt. It passes the STRING through unchanged while also emitting it to the UI.

To configure it:

1. Insert **AppBuilder Text Output** into the text path, for example `LLM → AppBuilder Text Output → CLIP`.
2. Add a **TEXT OUTPUT (Node)** control in Configure Panel.
3. Select the source Text Output node. Its node ID is also drawn directly on the node for easier identification.

Multiple Text Output nodes and controls can be used in one app. A result appears as soon as its node finishes and remains visible until the next generation starts.

### Better LoRA Stack editing

The LoRA Stack control was redesigned for longer, organized lists:

- strength accepts negative values from `-100` to `100`;
- `−` and `+` buttons adjust strength in steps of `0.05`;
- LoRAs can be enabled or disabled individually or with the master toggle;
- LoRA names are displayed without folder prefixes and the `.safetensors` extension, while the original full path remains stored;
- the picker is presented as a folder tree instead of one flat list;
- an **Edit** mode reveals selection, deletion, and drag handles so normal use stays uncluttered;
- drag and drop changes list order;
- a short, label-free divider line can visually separate groups such as styles and characters.

Divider entries are preserved in the control JSON but ignored by the Python LoRA loader, so they never affect generation.

### Controls resolved from workflow connections

A COMBO control can use **Auto from connection** instead of a manually copied option list. AppBuilder follows the matching Parameters Unpacker output to the connected workflow input and reads its declared options. This is useful for long or changing lists such as samplers and schedulers.

The resolver supports direct connections, reroutes, and supported subgraph paths. The resolved values are copied into the AppBuilder configuration so they remain available in standalone AppView. Use **Refresh** after changing the target connection. If one output reaches inputs with different option lists, Configure Panel reports a conflict instead of choosing one silently.

### Optional controls and visual groups

Controls that are rarely changed can be marked **Additional control** in Configure Panel. They are moved to one shared, collapsed **Additional controls** section at the bottom of AppView, with a count badge. This keeps the main panel compact without removing access to advanced settings.

A control can also receive a **Visual Group ID**. Consecutive controls with the same non-empty ID are enclosed by one subtle border, with no heading or large spacing. The ID is only a grouping key; it is not shown to the user. Groups are built separately in the main and Additional controls sections.

### Cache-safe UI updates

App Manager, AppView, and Configure Panel URLs include a timestamp cache buster, and the Python routes serve the main HTML pages with no-store headers. This prevents an old browser-cached interface from hiding newly installed frontend changes after an extension update.

## Local data and privacy

AppBuilder does not add a remote analytics or synchronization service. The added histories are browser-local:

| Data | Storage | Retention |
|---|---|---|
| Gallery media URLs | `localStorage` | Until cleared by the user/browser |
| Generation control snapshots | `localStorage` | Latest 200 generations; linked to gallery items |
| Prompt history | `localStorage` | 100 regular entries per field plus favorites |
| UI preferences | `localStorage` | Until cleared by the user/browser |

The saved media itself remains managed by ComfyUI. Clearing browser site data removes the local histories without deleting ComfyUI output files.

## Notes

- Final preview quality depends on the workflow producing a saved or previewable media output that ComfyUI reports through its normal execution events.
- Parameter restoration covers controls exposed by AppBuilder; it does not restore hidden workflow state.
- Browser-local history is intentionally lightweight and is not portable across browsers or devices.
- After updating the extension, restart ComfyUI and reopen AppView. The cache-busting changes should make a hard refresh unnecessary.

## Credits

- [ComfyUI](https://github.com/comfyanonymous/ComfyUI) by @comfyanonymous
- Original [ComfyUI-AppBuilder](https://github.com/lihaoyun6/AppBuilder) by @lihaoyun6
