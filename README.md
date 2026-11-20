# BeverlyLIB · Fluffy Hub UI

The default Fluffy Hub appearance uses Moss + Lime and a horizontal tab strip. Existing callers can still pass `TabLayout = "Side"`; the old palette is available in ThemeManager as **Legacy Mono**.

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
