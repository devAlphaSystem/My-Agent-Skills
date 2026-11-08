---
name: commandlog
description: Trigger this if the user says 'commandlog'. Every agent records each shell command it runs in a plain JSON file, with a timestamp, the expected outcome and the reason.
---

# Commandlog

Use this skill together with any other skill or task that runs delegates or shell commands, such as council, squad, debug, onboard, or qa. It adds one rule: before running a shell command, the agent records it in a JSON log. Commandlog changes no roles, models, workflow, or scope of the other skill.

## Log location

The log root is the `CommandLogs` folder on the current user's Desktop:

`<Desktop>\CommandLogs`

`<Desktop>` is the result of `[Environment]::GetFolderPath('Desktop')` in PowerShell. It is usually `%USERPROFILE%\Desktop`, but it can differ when the Desktop is redirected (for example, to OneDrive). Never hard-code a user name or a personal path. If the environment variable `COMMANDLOG_ROOT` is set, use its value as the log root instead.

Logs are grouped by workspace. Each workspace has its own folder, named after the last segment of the workspace root path, and one file inside it:

`<log root>\<workspace folder name>\commands.json`

Example: for the workspace `C:\Projects\My App`, the log is `<log root>\My App\commands.json`.

- The helper creates the workspace folder and the file when they do not exist.
- In a multi-project workspace, use the name of the workspace root folder that the user opened, not of each subproject.
- Characters not allowed in Windows folder names are replaced with `_`.
- If the log root cannot be created or written, report it to the user and stop before running any command.

## File format

`commands.json` is plain JSON: one array of objects, with no extra wrapper, comments, or metadata. Entries are in the order they were saved. Each entry has exactly four fields:

| Field             | Content                                                                           |
| ----------------- | --------------------------------------------------------------------------------- |
| `timestamp`       | Local date and time with UTC offset, ISO 8601. Set by the helper, never by agents |
| `command`         | The exact command, after redacting secrets                                        |
| `expectedOutcome` | One sentence on what the agent expects to happen after the command                |
| `reason`          | One sentence on why the agent needs to run it now                                 |

Example:

```json
[
  {
    "timestamp": "2026-11-08T12:41:07-03:00",
    "command": "npm run typecheck",
    "expectedOutcome": "TypeScript finishes with no errors, or lists the type errors.",
    "reason": "Check whether the bug comes from a type mismatch before reading more code."
  }
]
```

Do not add other fields, such as agent, directory, or result. Agents send only `command`, `expectedOutcome`, and `reason`. The helper adds `timestamp` when it saves the entry, which is just before the command runs.

## Language

Write everything in the log in English, including `expectedOutcome` and `reason`. This applies even when the user writes in another language. Keep each of the two fields to one short sentence, and make them specific to the task, not generic ("to check the code").

## What to record

Every agent records every shell command it runs, including the coordinator and all delegates. This covers builds, tests, linters, read-only Git commands, file reads and listings done through the shell, dev servers, and diagnostics.

Do not record:

- Calls to the logging helper itself
- Non-shell tool calls, such as the harness's file edit or search tools

## When to record

Record before running the command, not after. A command that crashes, hangs, or gets stopped must still appear in the log. Run the command only after the entry is saved. If saving fails, do not run the command. Report the failure to the coordinator.

## Bootstrap

Setting up the log is not logged. The coordinator does not record these steps, because the helper may not exist yet:

- Resolving the log root (for example, querying the Desktop folder)
- Creating the log root folder
- Creating or replacing `log_command.ps1`
- Running the helper once to test it

Every other shell command is recorded, starting with the first one after setup. The test run must use a separate temporary workspace name, so it does not add a test entry to a real `commands.json`.

## Secrets

Never write secret values to the log. Replace tokens, passwords, API keys, and similar values inside a command with `<redacted>`. Keep variable names and flags.

## Helper

The coordinator creates this helper once, as `log_command.ps1` directly inside the log root, if it does not exist. If it exists but differs from this version, the coordinator replaces it. It uses a named mutex and an atomic file replace, so parallel delegates can write to the same file without corrupting it.

```powershell
param([Parameter(Mandatory = $true)][string]$Workspace)

$ErrorActionPreference = 'Stop'
$root = $env:COMMANDLOG_ROOT
if (-not $root) {
  $root = Join-Path ([Environment]::GetFolderPath('Desktop')) 'CommandLogs'
}
$utf8 = New-Object System.Text.UTF8Encoding($false)
[Console]::InputEncoding = $utf8

$in = [Console]::In.ReadToEnd() | ConvertFrom-Json
foreach ($field in 'command', 'expectedOutcome', 'reason') {
  if (-not $in.$field) { throw "commandlog: missing field '$field'" }
}

$name = Split-Path -Leaf ($Workspace.TrimEnd('\', '/'))
$name = $name -replace '[\\/:*?"<>|]', '_'
if (-not $name) { $name = 'workspace' }

$dir = Join-Path $root $name
New-Item -ItemType Directory -Force -Path $dir | Out-Null
$file = Join-Path $dir 'commands.json'

$mutex = New-Object System.Threading.Mutex($false, "commandlog-$name")
if (-not $mutex.WaitOne(30000)) { throw 'commandlog: could not acquire lock' }
$tmp = "$file.tmp.$PID"
try {
  $list = New-Object System.Collections.ArrayList
  if (Test-Path -LiteralPath $file) {
    $text = [System.IO.File]::ReadAllText($file, $utf8)
    if ($text.Trim()) {
      foreach ($e in @($text | ConvertFrom-Json)) { [void]$list.Add($e) }
    }
  }

  [void]$list.Add([ordered]@{
    timestamp       = (Get-Date).ToString('yyyy-MM-ddTHH:mm:sszzz')
    command         = [string]$in.command
    expectedOutcome = [string]$in.expectedOutcome
    reason          = [string]$in.reason
  })

  $json = ConvertTo-Json -InputObject @($list) -Depth 5
  $json = [regex]::Replace(
    $json,
    '(?<!\\)((?:\\\\)*)\\u(003c|003e|0026|0027)',
    { param($m) $m.Groups[1].Value + [char][Convert]::ToInt32($m.Groups[2].Value, 16) }
  )

  [System.IO.File]::WriteAllText($tmp, $json, $utf8)
  if (Test-Path -LiteralPath $file) {
    [System.IO.File]::Replace($tmp, $file, [NullString]::Value)
  } else {
    [System.IO.File]::Move($tmp, $file)
  }
}
finally {
  if (Test-Path -LiteralPath $tmp) { Remove-Item -LiteralPath $tmp -Force }
  $mutex.ReleaseMutex()
}
```

Each agent records a command by piping the entry as JSON to the helper. Use a single-quoted here-string so PowerShell does not expand anything in the command text. Pass the real workspace root path in `-Workspace`:

```powershell
$root = if ($env:COMMANDLOG_ROOT) { $env:COMMANDLOG_ROOT } else { Join-Path ([Environment]::GetFolderPath('Desktop')) 'CommandLogs' }
@'
{
  "command": "npm run typecheck",
  "expectedOutcome": "TypeScript finishes with no errors, or lists the type errors.",
  "reason": "Check whether the bug comes from a type mismatch before reading more code."
}
'@ | powershell -NoProfile -File (Join-Path $root 'log_command.ps1') -Workspace "<workspace root path>"
```

If the agent runs in a shell without PowerShell, such as WSL, call the Windows executable (`powershell.exe`) with Windows-style paths. If neither is possible, report it to the user and stop.

## Briefing delegates

Each delegate starts fresh. The coordinator resolves the log root and includes in every delegate brief:

- The resolved log root, the helper path, and the workspace root path to pass in `-Workspace`
- The invocation example
- The three fields agents send, the note that the helper adds `timestamp`, and the one-sentence rule for `expectedOutcome` and `reason`
- The rule that all log text is written in English
- The rule to record before running, and the secrets rule

## Permissions

This skill adds one exception to the other skills' file rules. Every agent may write inside the log root (workspace folders, `commands.json`, and its temporary file), and the coordinator may create or update the helper script. Nothing else outside the workspace may be written. The Git restriction of the other skills still applies. Do not run the helper through Git or any version-control command.

Agents never edit, reorder, or delete earlier entries. Only append.

## Final report

In the final reply, the coordinator states:

- The path of the log file
- The number of entries recorded in the run
- Any command that could not be recorded and why

## Scope

Commandlog only records commands. It does not review, block, or approve them, and it does not change which commands the other skill allows.
