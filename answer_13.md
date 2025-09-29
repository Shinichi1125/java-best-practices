This is a classic Windows “file is in use” problem during `mvn clean`. Maven tries to delete `target/` recursively; if **any file is locked** by another process, you’ll get:

> Failed to clean project: Failed to delete: …\target\…\something.bpmn

Why it happens (most common → less common):

1. A process still holds the file

* Your app is still running (from a previous Run/Debug).
* A test forked JVM didn’t exit cleanly.
* VS Code/IntelliJ test runner or Java language server indexing `target`.
* AV/backup/indexer (Defender/OneDrive) scanning `target`.
* File preview open in Explorer.

2. File is marked read-only or path is too long

* Read-only attribute on the file.
* Deep path under `target` hitting Windows path limits (less likely, but possible).

3. Permissions glitch

* Working dir on a synced/controlled folder (OneDrive) that locked it.

## Quick fixes (in order)

1. Kill anything using the project:

* Stop all Run/Debug sessions.
* Close any app that might watch `target` (test runner, code coverage, file preview).
* On Windows, find the locker:

  * Open **Resource Monitor** → CPU tab → **Associated Handles** search for the filename.
  * Or use **Process Explorer** (Sysinternals) → Find Handle (`Ctrl+F`) and close the handle/kill the process.

2. Manually remove `target`

* PowerShell:

  ```powershell
  Stop-Process -Name java -ErrorAction SilentlyContinue   # if you’re sure no other java apps needed
  attrib -R -S -H -A .\target\* -Recurse
  Remove-Item -Recurse -Force .\target
  ```
* CMD:

  ```cmd
  attrib -R -S -H -A target\* /S
  rmdir /S /Q target
  ```

3. Retry build

```bash
mvn clean package
```

## If it keeps happening

* Exclude your project’s `target/` from antivirus/OneDrive.
* In VS Code/IntelliJ, disable “build automatically”/background test discovery for this project.
* Shorten the project path (avoid very deep directories).
* As a temporary workaround:

  ```bash
  mvn clean -Dmaven.clean.failOnError=false package
  ```

  (Not ideal—better to fix the lock.)

## Why a BPMN under `target/`?

Resources (like `.bpmn`) are copied from `src/main/resources` into `target/classes` during build; that’s what Maven tries to delete. The source file isn’t the issue; the **copied** one is locked.

If you want, paste the exact file path and I’ll suggest which tool/process is most likely locking it (and how to release it).
