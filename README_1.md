# Computer Attendance & Monitoring System

Two parts:
1. **Agent** (`agent/`) - small Windows program on each PC. Every 60 s it sends: PC name, optional employee ID, "online", and *seconds since last keyboard/mouse input*. Nothing else (no screenshots, keystrokes, window titles, files, passwords).
2. **Google Sheet backend** (`google-apps-script/`) - receives heartbeats, updates attendance, dashboard, search, notices and alerts.

## 1. Set up the Google Sheet (10 min)
1. Create a new Google Sheet > **Extensions > Apps Script**.
2. Paste `Code.gs` into the editor. Project Settings > tick "Show appsscript.json" and paste `appsscript.json`.
3. Run **setup** once (authorize). It creates: Settings, Employees, Holidays, Heartbeats, Search, Attendance Dashboard, today's "dd-MM-yyyy Attendance" sheet, and the 5-minute + daily triggers.
4. **Deploy > New deployment > Web app** - Execute as *Me*, Access *Anyone*. Copy the Web app URL.
5. In **Settings** fill Notification Email, working hours, threshold, working days. Copy the **API Token**.
6. In **Employees** enter ID, name, department and the **PC name** (Windows hostname) for each employee. Add holidays in **Holidays**.

## 2. Install the agent on each PC
1. On one PC with Python: edit `agent/config.json` (web_app_url + token), run `build_exe.bat`.
2. Copy `dist\AttendanceAgent.exe` + `config.json` + `install.bat` to each PC, run `install.bat` as administrator. It starts at every login.
3. Check what is sent: `AttendanceAgent.exe --show-data` (needs console build) or `python attendance_agent.py --show-data`.
4. Within a minute the PC appears in the sheet (Heartbeats tab, then the daily sheet).

## Status rules (all configurable in Settings)
| Status | Rule |
|---|---|
| Active | heartbeat received and input within the inactivity threshold |
| Inactive | no input >= Inactivity Threshold (default 15 min) - notice written |
| Not Using Computer | PC online, no input >= Escalation Threshold (default 30 min) - email/Chat alert |
| Computer Offline | no heartbeat for Offline After minutes (default 5) |
| No Login / Absent | no heartbeat today after Start Time + Grace |
| Late | first login after Start Time + Grace |

Statuses refresh every 5 minutes. Alerts are sent once per employee per status per day.

## Notes
- Must run in the user's session (Scheduled Task at logon), not as a Windows service, otherwise idle time can't be read.
- Idle time = input only. Watching a video or reading a long document with no mouse/keyboard counts as idle - choose thresholds accordingly and use the Remarks column for exceptions.
- Tell employees what is collected (see --show-data) and get it approved by HR/management; monitoring policies/consent requirements apply.
- Windows only. macOS/Linux agents would need a different idle-time call.
