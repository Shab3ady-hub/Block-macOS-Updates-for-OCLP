# Block-macOS-Updates-for-OCLP
A free, open-source app that blocks macOS updates and removes the “update to the new macOS” banner and the red badge. Everything is backed up first, and there’s a one-click undo.

BLOCK macOS UPDATES
Stops a Mac from finding and downloading macOS updates, and removes the "update to the new macOS" banner

If this tool helped you, please consider helping by donating to me on PayPal 

READ THIS FIRST
 - This blocks ALL macOS updates, security updates included, not only the next big upgrade. It is meant for
   people who must stay on their current macOS (for example Macs running OpenCore Legacy Patcher, where an
   accidental upgrade can break the install). If you want security updates, do not use it.
 - It edits /etc/hosts and system preferences as an administrator, and at the end it DELETES any macOS update
   that is already downloaded. That deletion cannot be undone. Everything else can be undone with "Undo the block".
 - It is provided "as is", with no warranty. Use it at your own risk (see LICENSE.txt).
 - The app is not signed or notarized by Apple (see "Opening the app").

SUPPORTED macOS VERSIONS
  Ventura 13 ........................ VERIFIED on a real Mac (MacBook Air 2015, OCLP): the banner went away.
  Sequoia 15 ........................ PARTLY VERIFIED on a real Mac: the banner went away; the red badge in
                                      System Settings may need a restart, or may stay.
  Sierra 10.12 to Monterey 12,
  Sonoma 14, Tahoe 26 ............... NOT TESTED. The app is built for them, but each macOS stores and downloads
                                      updates differently. "Check block status" and the log show what failed.
  Apple Silicon Macs ................ NOT TESTED.
  Stock Macs with SIP on ............ NOT TESTED. Deleting downloaded updates may be refused (the app says so);
                                      the other changes are in places SIP does not protect.

OPENING THE APP
Keep "Block macOS Updates.app" and this README in the same folder.
The app is not signed by Apple, so macOS may warn you the first time:
 - Right-click the app > Open. On Sequoia and Tahoe this is usually not enough: open System Settings >
   Privacy & Security and click "Open Anyway" after the first attempt.
 - Files copied by USB drive usually open without a warning.

THE MENU
  Block macOS updates   Asks you to confirm (Cancel / Block), then asks for your password and shows a "Working..."
                        window. At the end it tells you whether it worked, in plain words; "Open log" shows
                        every detail. If the Mac can still reach Apple (VPN, proxy, custom DNS) it does NOT say
                        "done": it tells you the banner may stay.
  Check block status    Checks (needs your password) that the block still works: /etc/hosts entries, Apple
                        servers unreachable, automatic checks off, no data about a newer macOS back in the
                        cache, and no downloaded update waiting. If something broke, "Repair" applies the block
                        again. Use it if the banner comes back.
  Undo the block        Restores what the app changed. With one backup it asks to confirm; with several it
                        offers "Undo the last block" or "Undo all blocks". Deleted downloads cannot come back.
  Copy debug report     Copies a report to your clipboard (and saves it in ~/Library/Logs) to paste into a bug
                        report. It has no name, serial number or network details.
  Open the guide        Opens this README.
After blocking, RESTART the Mac before opening System Settings (or System Preferences).
Afterwards, checking for updates will fail to connect or say "invalid URL": that is normal, the block works.

WHAT IT CHANGES
 1. Turns off automatic update checks, downloads and installs, and clears the red badge: it forgets the saved
    "updates available" list and restarts the Dock and the update notifier.
 2. Blocks swscan.apple.com, swdist.apple.com and gdmf.apple.com in /etc/hosts, over IPv4 and IPv6.
    (Blocking only IPv4 is not enough: the Mac kept reaching Apple over IPv6.)
 3. MOVES (does not delete) the cached update data, and the update service's saved-state file (where it remembers
    pending updates), into the backup folder, only after confirming that the servers are really blocked.
 4. At the end it clears updates that are already downloaded: it DELETES what is in /Library/Updates, the folder
    "/macOS Install Data" and the downloaded update files (.asset) in /System/Library/AssetsV2, then restarts the
    macOS update service so that it forgets any update it thought was pending. If macOS refuses something (SIP) or
    will not restart the service, it says so and carries on; restarting the Mac does the same.
Other Apple servers (such as mesu.apple.com) are NOT blocked on purpose: blocking them can affect other Apple
downloads. A link on apple.com that downloads macOS can still bring the banner back by another route; if it
does, use "Check block status" > "Repair" and restart.

UNDOING
Use "Undo the block". Backups and logs are in /Library/macOS-Updates-Block-Backup/<date-time>/.
The restore puts back /etc/hosts, the preferences and the moved cache data and saved-state file. It cannot bring back downloaded
updates that were deleted.

TERMINAL
  sudo bash "/path/to/Block macOS Updates.app/Contents/Resources/block-macos-updates.sh" [options]
  --aggressive   also disable the softwareupdated service (more invasive; usually not needed)
  --keep-cache   do not move the cached update data
  --status       only check the block (changes nothing); exit code 4 means problems
To undo: sudo bash /Library/macOS-Updates-Block-Backup/<date-time>/restore-update-settings.sh (use a folder
that has no RESTORED file: that backup was already restored).

TROUBLESHOOTING
 - The banner came back (for example after opening an Apple macOS download link): "Check block status" >
   "Repair", then restart before opening System Settings.
 - A "Restart to update" button or an update listed as downloaded: do NOT click it. "Repair" deletes it.
   OCLP users who get a "system version mismatch" error while root patching have this situation. If the error
   stays after "Repair", OCLP's own documentation describes a PurgePendingUpdate tool for it; it is not part of
   this package.
 - The red badge on Settings is still there after a restart: run this read-only command and send the output with
   your bug report:
     { sudo defaults read /Library/Preferences/com.apple.SoftwareUpdate | grep -E "LastUpdatesAvailable|RecommendedUpdates"; defaults read com.apple.systempreferences AttentionPrefBundleIDs; launchctl list | grep -i softwareupdate; }
 - "Operation not permitted" in the log: give the app Full Disk Access (System Settings > Privacy & Security).
 - Managed (MDM) Macs and some VPNs can overwrite /etc/hosts; the status check will say the block is missing.
 - The log is in English and lives in ~/Library/Logs (BlockMacOSUpdates-*.txt).

BUG REPORTS
Use "Copy debug report" and paste it into your report, together with your macOS version and what you saw and send it to reddit user u/Shab3ady
