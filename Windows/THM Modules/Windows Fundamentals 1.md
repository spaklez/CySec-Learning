## Windows OS Overview

Everyone knows about Windows, it's one of the most widely used operating systems out there, which also makes it a big target for hackers and malware writers.

Windows XP was one of Windows' most popular versions, then came Vista, which brought a major overhaul to the OS.

One difference between the Pro and Home versions of Windows is BitLocker Drive Encryption. BitLocker is Microsoft's own full disk encryption technology, built into Windows since Vista. It encrypts an entire volume so that if someone pulls the disk or steals the laptop, the data on it is unreadable without the key or recovery password.

## The Windows Desktop (GUI)

This part is pretty straightforward, Windows has these features:

- **Desktop**: icons for programs, folders and files, shortcuts for quick access. Right click gives you options to resize icons, arrange them, and create new items. Display settings and Personalize (wallpaper, theme, colors) are both accessed from here.
- **Start Menu**: opens from the Windows logo bottom left. Shows account shortcuts (Documents, Pictures, Settings, power options) at the top, an alphabetical list of installed apps in the middle, and pinned tiles on the right.
- **Search Box: search for apps, files, and settings.
- **Task View**: shows open windows and virtual desktops.
- **Taskbar**: shows open apps, with preview thumbnails on hover. Right click gives you options for toolbars and other taskbar settings.
- **Toolbars**: optional extras that can be enabled on the taskbar.
- **Notification Area**: bottom right, shows the clock, volume, network icon, and other background app icons. Customizable in Taskbar settings.

## NTFS and Clusters

Windows uses NTFS (New Technology File System). NTFS has a feature called self-healing NTFS, which automatically detects and repairs minor file system corruption in the background without needing to take the volume offline. For bigger corruption, `chkdsk` can scan and repair NTFS volumes while trying to keep them online, to minimize downtime.

NTFS volumes can be as large as 8 petabytes (PB). Earlier versions of Windows only support volumes up to 256 terabytes (TB). The actual max volume and file size depends on the cluster size and the number of clusters NTFS can address (up to 2^32 - 1 clusters). A cluster is the smallest unit of disk space the file system can allocate to a file.

The table in the room shows the trade-off: smaller clusters mean less wasted space per file, but a hard ceiling on how large a volume or file can get, because NTFS only has a fixed number of addressable clusters. Larger clusters support much bigger volumes, but waste more space, especially with lots of small files.

So basically, larger clusters mean more wasted space per file (a small file still eats a whole cluster) Additionally NTFS just track things sector by sector instead of grouping into cluster, that's because 
Every allocable unit needs an entry in NTFS's metadata, the Master File Table (MFT) tracks which clusters belong to which file. If NTFS tracked individual 512-byte sectors instead of 4KB clusters, a 1TB drive would be around 2 billion sectors to address, versus about 256 million clusters at 4KB. 

You'd also need bigger addresses/pointers to reference each unit, and files would be more prone to fragmentation since they'd get split across way more discrete pieces to track and seek between. 

## FAT vs NTFS

Before NTFS there was FAT16/FAT32 (File Allocation Table) and HPFS (High Performance File System). FAT partitions are still around today, mostly on USB devices and MicroSD cards, but not usually on Windows computers, laptops, or servers anymore.

NTFS is a journaling file system, meaning it logs its tasks so that if there's an unexpected shutdown, the file system can automatically repair folders and files on disk using the log. FAT file systems can't do this.

## NTFS Permissions

NTFS volumes let you set permissions on files and folders to grant or deny access. The permissions are:

- Full control
- Modify
- Read & Execute
- List folder contents
- Read
- Write

## Alternate Data Streams (ADS)

Alternate Data Streams (ADS) is a file attribute specific to NTFS. Every file has at least one data stream by default (`$DATA`), which is the actual content you see when you open the file. ADS lets a file carry additional, hidden streams of data attached to that same file, without changing the file's visible size or content in Explorer.

Windows Explorer doesn't show these extra streams natively, they're invisible unless you go looking for them. You can view and interact with ADS a few ways:

- **PowerShell**: `Get-Item -Path file.txt -Stream *` lists all streams on a file. `Get-Content -Path file.txt -Stream streamname` reads the content of a specific stream. `Set-Content` and `Add-Content` also support the `-Stream` parameter to write to one.
- **cmd**: `dir /r` shows streams in a directory listing.
- There are also third-party tools built specifically for browsing ADS.

This is exactly why ADS gets abused. Because Explorer hides these streams by default and most users, and a lot of basic file scans, never think to check for them, malware writers have used ADS to hide payloads or data inside an otherwise normal-looking file. The file's name, icon, and size in Explorer stay the same, the hidden stream just rides along attached to it.

## System Files: C:\Windows, %windir%, System32

Inside the C drive, `C:\Windows` is traditionally the folder that holds the Windows OS, though it doesn't have to be on the C drive, and technically doesn't even have to be called `Windows`. Like Linux, Windows uses environment variables, and the one for the Windows directory is `%windir%`.

Inside the Windows folder is `System32`, which is crucial for the OS to function, it holds important system files and libraries. This folder has a lot of DLL and EXE files. DLLs (Dynamic Link Libraries) are shared library files used by Windows programs, EXEs in System32 are the actual Windows system utilities. For example, opening Task Manager runs `Taskmgr.exe`, which lives in System32.

## User Accounts

A local Windows system has two account types: Administrator and Standard User.

- Administrator can make system-level changes: add/delete users, modify groups, change system settings, and so on.
- Standard User can only change files/folders that belong to them, and can't do system-level changes like installing programs.

When an account is created, Windows makes a profile for it under `C:\Users`. Every user profile gets the same set of folders: Desktop, Documents, Downloads, Music, Pictures.

Most local users end up logged in as local administrators day to day. The problem with that is the more privileged the account you're logged into, the more that can go wrong. Running everything as admin all the time removes the safety margin that a standard account gives you. To cut down on that risk, Microsoft introduced User Account Control (UAC).

UAC works by not giving even an administrator account full elevated rights by default in a normal session. When something needs higher-level privileges to run, you get a prompt asking you to confirm it.

## Task Manager

Task Manager shows what's running on the system and how much of the machine it's using. A few general points:

- **Processes tab**: every running app and background process, with live CPU, memory, disk, and network usage per process. This is the tab to use for spotting what's hogging resources or ending a frozen app.
- **Performance tab**: overall system graphs for CPU, memory, disk, network, and GPU if present.
- **App history**: resource usage totals over time for Store apps.
- **Startup tab**: lists what launches automatically at boot and its relative impact on startup time, and lets you disable entries.
- **Users tab**: resource usage broken down per logged-in user, useful on shared or RDP machines.
- **Details tab**: a lower-level view of processes, closer to something like `tasklist` on the command line, includes PID, status, and priority.
- **Services tab**: shows Windows services and lets you start/stop/restart them.

It's one of the first places to look when something is running slow, hanging, or using more CPU/memory/network than expected.

Well that was WF1, pretty straightforward but kind of a good refresher.