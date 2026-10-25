# Windows Troubleshooting Guide

Started this while I was working in IT support. Just wanted a place to dump all the Windows issues I deal with so I don't keep forgetting them.

Slow PCs, Windows Update failing, startup problems, apps crashing, driver issues — that kind of stuff. I add to it when I run into something new or find a better way to fix something. Same idea as my network guide.

## What's in here

- WIN-001: Computer is slow
- WIN-002: Windows Update fails
- WIN-003: Windows won't start normally
- WIN-004: Application crashes
- WIN-005: Device driver problems

---

## WIN-001: Computer Is Slow

System Performance

User says their computer takes forever to start, apps are slow, Windows freezes sometimes. This is honestly the #1 complaint I get. Half the time it's just one app eating all the CPU, the other half the user has like 40 browser tabs open and wonders why things are slow.

What usually causes it (in order of how often I see it):

- High CPU/memory/disk usage
- Way too many startup apps
- Disk almost full
- Updates or AV scans running in background
- Bad drivers
- Overheating or hardware failing
- Specific app being broken

So first thing — I ask the user some questions. When did it start? Is it the whole PC or just one app? Did they install anything recently? You'd be surprised how often the answer is "yeah I installed something two days ago" and that's the whole story.

Then Task Manager (Ctrl + Shift + Esc). Look at CPU, Memory, Disk. If something is at 90% CPU the rest of the diagnosis doesn't really matter — that's your problem right there.

After that, storage. Settings → System → Storage. Below 10% free on the system drive is usually bad. Windows gets weird when C: is full.

Last thing I check — Reliability Monitor:

```
perfmon /rel
```

Underrated tool honestly. Shows you a timeline of crashes, updates, all that stuff. When the user says "it got slow last week" you can literally see what happened last week. Pretty useful.

Fixing it depends on what you found:

- High resource usage → kill or update the process hogging things.
- Too many startup apps → disable the ones that don't need to start with Windows.
- Low disk space → clean it up using whatever's approved in your org.
- Pending updates → let them finish, restart.
- Overheating or hardware → escalate, this isn't a software fix.

Don't end processes you don't recognize. Some of them look weird but they're actually critical Windows services. I made that mistake once, took me an hour to figure out why nothing worked after.

To verify — ask the user to try the thing that was slow. Check Task Manager again. Better? Good. Not better? Keep digging.

---

## WIN-002: Windows Update Fails

OS Maintenance

Windows Update is stuck, fails, or throws an error while downloading/installing. Annoying because users always want updates done but then complain when they take forever.

Usual suspects (most common first):

- Not enough free disk space
- Internet connection issues
- Wrong date/time
- Restart pending
- Windows Update components corrupted
- Incompatible drivers or software
- Company policy blocking updates

Diagnosis — open Windows Update in Settings. If there's an error code, write it down. That code is basically your main clue. Microsoft Learn has docs for most of them.

Then check the basics:

- Is the internet actually working?
- Is the date and time correct? Wrong time breaks HTTPS and Windows Update silently. People never suspect this.
- Free disk space?
- Did Windows ask for a restart? Just restart it. A lot of "stuck" updates just want a restart.

Look at update history — sometimes one specific update fails and the rest install fine. That tells you it's not a generic Windows Update issue, it's that specific update.

If the PC is company-managed, check if there are update policies. Deferral rings, WSUS, Intune update policies — any of these can be the actual cause and you can't override them locally.

Try the built-in Windows Update troubleshooter. It's not magic but it fixes more cases than you'd expect.

Solutions:

- Bad connection → fix the internet first.
- Wrong time → fix it.
- Low disk space → clean up.
- Pending restart → restart.
- Specific error code → look it up on Microsoft Learn and follow their steps.
- Company policy → talk to whoever manages updates.

Don't run "Windows Update reset scripts" you find on random blogs. I've seen these break things worse than the original problem. The official Microsoft steps are slower but safer.

To verify — try the update again, check history to see if it succeeded.

---

## WIN-003: Windows Does Not Start Normally

Boot and Recovery

PC won't boot, keeps restarting, blue screen, or it drops into Windows Recovery Environment. These are the calls I hate the most honestly. User is stressed, their data might be at risk, one wrong move makes things worse.

What's usually going on:

- Corrupted system files
- Bad update
- Driver problem
- Recent hardware or software change
- Boot config messed up
- SSD or HDD failing

First — write down the exact error message or stop code. Blue screen codes tell you a lot:

- `IRQL_NOT_LESS_OR_EQUAL` → almost always a driver.
- `INACCESSIBLE_BOOT_DEVICE` → storage or storage driver.
- `CRITICAL_PROCESS_DIED` → could be a lot of things, check the dump.

When does it happen?

- Before Windows logo → boot or hardware.
- During startup → driver or service.
- After login → startup app or service.
- Right after an update or driver install → that's your suspect.

Disconnect USB stuff that isn't needed. Printers, docks, external drives — anything. Try booting. I once spent two hours on a boot issue that was just a bad USB dock. Felt pretty dumb after.

Open WinRE if you can get to it. Useful options there:

- Startup Repair
- Safe Mode
- Uninstall recent update (if that's the cause)
- System Restore (if you have a restore point)

Before doing anything major — is the user's data backed up? If BitLocker is on, do you have the recovery key? Windows will ask for it during recovery operations that touch boot config. Without the key you're stuck.

Fixing it — start with the least destructive option that matches the problem:

- Bad update → uninstall it.
- Bad driver → roll back.
- Boot issue → Startup Repair.
- System file corruption → SFC /scannow, DISM if needed.
- Hardware failure → don't try to fix it in software, investigate the hardware.

Reinstalling Windows is NOT the first thing to try. I know it's tempting. Don't. Check data, check recovery options, then maybe.

Verify — Windows boots normally, survives a restart, user can log in and access their stuff.

---

## WIN-004: Application Crashes

Application Troubleshooting

App closes by itself, freezes, or throws errors while the user is working.

What's usually wrong:

- App bug
- Outdated app version
- Corrupted settings
- Not enough RAM/disk
- Missing dependency
- Compatibility issue
- Damaged app files

Get details first:

- Which app?
- What were you doing when it crashed?
- Was there an error message?
- When did this start?
- Just you or other users too?

Get the app version and steps to reproduce. If you can reproduce it you're halfway there.

Try to reproduce it. If it crashes on demand that's actually good — you can test fixes immediately instead of waiting for it to happen on its own.

Reliability Monitor:

```
perfmon /rel
```

Look for app failures around the time of the crash.

Then Event Viewer:

```
eventvwr.msc
```

Windows Logs → Application. Look for errors related to the crash. Usually includes the faulting module name — that tells you what actually broke. Way more useful than the generic "app stopped working" message the user sees.

Check the app environment too. Pending updates? Missing dependencies? Disk full? RAM maxed out?

Fixes:

- Outdated → install the update.
- Damaged install → use the app's repair option if it has one.
- Corrupted settings → reset them (back up first if needed).
- Still broken → reinstall per vendor's instructions.
- Keeps crashing → collect logs and escalate to the app's support team.

Don't delete app data folders without checking what's in them. Users keep the weirdest stuff in there.

Verify — try the same workflow that crashed. Works? Done. Still broken? Check Reliability Monitor and Event Viewer again.

---

## WIN-005: Device Driver Problem

Hardware and Device Management

A device — network adapter, printer, audio, graphics, whatever — is missing from Device Manager, has a yellow warning, or just doesn't work.

What's usually wrong:

- Missing or corrupted driver
- Recent driver update broke things
- Incompatible driver version
- Device is disabled
- Hardware not supported
- Hardware actually failing

Open Device Manager:

```
devmgmt.msc
```

Find the device. Yellow triangle with exclamation mark = something's wrong.

Open Properties, look at Device status. There's usually an error code. Knowing the codes saves a lot of guessing:

- Code 28 → driver not installed.
- Code 43 → device reported a problem (often driver, sometimes hardware).
- Code 10 → device can't start.
- Code 31 → driver not working.

Write down the device name and model. If you need the hardware ID, Details tab → Hardware Ids. That's what you search for when looking for a driver.

Ask the user — did this start after a Windows update? After they installed a new driver? After a hardware change? Recent changes are usually the cause.

Get the driver from Windows Update or the manufacturer's official website. Emphasis on official. Third-party driver sites are full of bundled garbage and sometimes the "driver" isn't even the right one.

Fixes:

- Device disabled → enable it (if it was disabled by mistake and you're authorized).
- Missing/incompatible driver → install the right one from an approved source.
- Recent driver update caused it → roll back the driver.
- Hardware fault → follow the manufacturer's diagnostic steps.
- Still broken → document the error code and escalate.

Don't install drivers from random sites. Seriously. I've seen "driver updater" tools install adware alongside the actual driver.

Verify — check Device Manager again, status should be normal. Then actually test the device. Print something, play audio, connect to the network. Clean status in Device Manager doesn't always mean the device works for real.

---

## Tools I Use

| Tool | What it does | How to open |
|------|---|---|
| Task Manager | CPU/memory/disk usage, startup apps | Ctrl + Shift + Esc |
| Device Manager | Hardware and drivers | `devmgmt.msc` |
| Reliability Monitor | Crash and update history | `perfmon /rel` |
| Event Viewer | Windows and app events | `eventvwr.msc` |
| Windows Update | Update status | Settings → Windows Update |

These tell you what's actually happening before you start changing things. Error messages point you in a direction but you usually need more than one check to confirm the cause.

---

## General Checklist

- Figure out what's actually broken and how it affects the user.
- Is it one app, one device, or the whole system?
- Recent changes? Updates? New hardware? New software?
- Get error messages and logs before changing anything.
- Check resource usage and device status if relevant.
- One change at a time. Write down what you did.
- Back up important data before anything destructive.
- Test the original problem after the fix.
- Resolved? Document it. Not resolved? Document that too.
- Escalate when you don't have the access or knowledge.

