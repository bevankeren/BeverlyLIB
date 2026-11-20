# BeverlyLIB · Fluffy Hub UI

The default Fluffy Hub appearance is **Graphite Silver**: charcoal surfaces, muted silver controls, a centered search field (`Left Ctrl+K` while the window is open), and a slim icon rail for main navigation. The rail uses each module's icon, or its initial when no icon is available. Explicit `AddCheckbox` controls are unchanged; `AddToggle` now uses a switch. On wide rows, labeled dropdowns sit on the right; narrow rows fall back to stacked fields.

Existing callers can still use `TabLayout = "Side"`. In top layout, `IconRail = false` restores the horizontal list of main tabs. Tabs with content on one side use the full width; tabs with both sides populated keep two columns. Pass `AdaptiveColumns = false` to preserve two columns even when one side is empty. Saved themes are not overwritten; choose **Graphite Silver** in ThemeManager to switch back to it. **Moss + Lime** and **Legacy Mono** are still available.

## Contextual navigation

With the icon rail enabled, `Window:AddTab` creates a module on the left. Use `module:AddSubTab` for its sections. The top strip shows only the active module's sections and is hidden when fewer than two sections are visible. Switching modules preserves each module's last visible section for the life of that window instance. If a selected section is hidden or removed, the module falls back to its first visible section or its own page.

```luau
local Home = Window:AddTab("Home", "house")
local Presets = Window:AddTab("Presets", "layers")
local Settings = Window:AddTab("Settings", "settings")

local Saved = Presets:AddSubTab("Saved")
local Transfer = Presets:AddSubTab("Import / Export")
local Appearance = Settings:AddSubTab("Appearance")
local Input = Settings:AddSubTab("Input")
local About = Settings:AddSubTab("About")

-- Add components directly to Home, and to each sub-tab for grouped modules.
```

## Fluffy Pulse

The status strip starts at **Idle** and the waveform flashes only when the owning script reports real work. Call the matching function from your script's callbacks instead of sending heartbeats from a UI timer.

```luau
local Window = Library:CreateWindow({
    Title = "Fluffy Hub",
    ScriptHeartbeatTimeout = 8, -- seconds; default 6, clamped to 1–120
})

local function onStartOrResume()
    Window:SetScriptStatus("Running")
end

local function onWorkCheckpoint(tasksDone: number)
    Window:ReportScriptHeartbeat(("%d tasks done"):format(tasksDone))
end

local function onPause()
    Window:SetScriptStatus("Paused", "Waiting for input")
end

local function onStop()
    Window:SetScriptStatus("Stopped")
end

local function onError(message: string)
    Window:SetScriptStatus("Error", message)
end
```

If a running script stops reporting, the strip changes to **Stale · No heartbeat**; the next report brings it back to **Running**. Reports cannot override a deliberate pause, stop, or error. `Window:GetScriptStatus()` returns a snapshot with `State`, optional `Detail`, `Heartbeats`, and `LastHeartbeatAge` in seconds (absent before the first report). Starting or explicitly resuming a run resets the counter; automatic recovery from Stale does not.

Set `ScriptStatus = false` to hide the strip and minimized status text without disabling tracking. Set `Animations = { ScriptPulse = false }` for a static trace. The library displays reports from its caller; it does not inspect or guess whether unrelated scripts are running.

Upstream component documentation: https://docs.mspaint.cc/obsidian
