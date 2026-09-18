## System Configuration (msconfig)

The System Configuration utility (msconfig) is for finding and isolating issues. When you use this utility, you can select options to temporarily prevent services and programs from loading during the Windows startup process. Doing this, you can start Windows and disable and enable processes one at a time, pinpointing the service causing the issue.

The utility has five tabs: General, Boot, Services, Startup, and Tools.

In the General tab, we can select what devices and services for Windows to load upon boot. The options are: Normal, Diagnostic, or Selective.

In the Boot tab, we can define various boot options for the Operating System.

The Services tab lists all services configured for the system regardless of their state.

The Startup tab however is empty, as Windows itself advises using Task Manager (taskmgr) to manage (enable/disable) startup items. Another way to view startup programs in Windows is by typing shell:startup in the Run dialog. This will display all startup programs as shortcuts or executables that are configured to run automatically the next time a user logs in.

For the Tools tab, there is a list of various utilities that we can run to configure the operating system further.

More configuration options can be found in the System Properties panel.

## Computer Management (compmgmt)

Another useful tool available from the System Configuration panel is Computer Management (compmgmt). This utility has three primary sections: System Tools, Storage, and Services and Applications.

One tool under System Tools is Task Scheduler, which lets us create and manage tasks that the computer will carry out automatically at set times. A task can run an application or a script, and can be configured to run at log in, at log off, or on a specific schedule, like every five minutes. To view the scheduled tasks present on the system, click Task Scheduler Library, which displays all the scheduled tasks on the system.

Another tool is Event Viewer, which lets us view events that have occurred on the computer. These records act as an audit trail that can be used to understand the computer's activity, and are often used to diagnose problems and investigate actions executed on the system. There are five types of events that can be logged.

Performance Monitor (perfmon) is used to view performance data either in real time or from a log file.

Device Manager allows us to view and configure the hardware, such as disabling any hardware attached to the computer.

Under Storage is Windows Server Backup and Disk Management. Disk Management is a system utility that enables you to perform advanced storage tasks, including:

- Set up a new drive
- Extend a partition
- Shrink a partition
- Assign or change a drive letter (ex. E:)

You can see all the services and their statuses by clicking the Services button under Services and Applications.

## System Information (msinfo32)

Another tool available from the System Configuration panel is System Information (msinfo32). This tool gathers information about your computer and displays a comprehensive view of your hardware, system components, and software environment. The information in System Summary is divided into three sections: Hardware Resources, Components, and Software Environment.

System Summary itself is just the general specs, stuff like your processor brand and model. 

Hardware Resources is more low level than the average user needs, basically bus addresses and driver level stuff. Components shows details on the actual hardware installed. 

Software Environment covers what's running plus software you've installed, and this is also where Environment Variables and Network Connections show up, which ties back to what we touched on in WF1 with the System32 folder, environment variables just store OS level info like the system path, how many processors the OS is using, and where temp folders live. 

You can view the same environment variables straight from msinfo32, or the more familiar route through Control Panel > System and Security > System > Advanced system settings > Environment Variables, or Settings > System > About > System info > Advanced system settings > Environment Variables. 
## Resource Monitor (resmon)

Another tool available from the System Configuration panel is Resource Monitor (resmon).

Resource Monitor gives you a live per-process and overall breakdown of CPU, memory, disk, and network usage, and it'll also tell you which processes are holding onto specific file handles and modules.

You can filter down to just the processes you care about, and from there start, stop, pause, or resume services, or kill an app that's stopped responding, straight from the interface. It also has a process analysis feature built in that can spot deadlocked processes and file locking conflicts, which is useful because it means you can actually try to resolve the conflict instead of just force closing the app and losing whatever you were working on.

In the Overview tab, Resmon splits into four sections: CPU, Disk, Network, and Memory, each one showing live usage for that resource so you can see straight away what's actually driving the load.

## Command Prompt Basics

The command prompt (cmd) is another tool that's part of the System Configuration panel.

- `hostname` outputs the computer's name.
- `whoami` outputs the name of the currently logged in user.
- `ipconfig` shows the network address settings for the machine.
- Most commands have a help manual you can pull up with `/?`, so `ipconfig /?` shows the syntax and extra parameters for ipconfig specifically.
- `cls` clears the command prompt screen.
- `netstat` shows protocol statistics and the current TCP/IP connections. It can be run alone or with parameters like `-a`, `-b`, `-e`, and the output changes depending on which ones you add.
- `net` is used to manage network resources and works through sub-commands. Typing `net` on its own shows the syntax and a few of the available sub-commands.
- For `net`, `/?` doesn't pull up the help manual, you need `net help` instead, so for example `net help user` gives the help info for the user sub-command specifically. Same pattern works for other net sub-commands like `localgroup`, `use`, `share`, and `session`.

## Windows Registry

Another tool available from the System Configuration panel is the Windows Registry. It's basically a central hierarchical database Windows uses to store the configuration info it needs for the OS itself, for individual users, for installed applications, and for hardware devices.

There are various ways to view/edit the registry. One way is to use the Registry Editor (regedit).

Basically everything from user settings to driver configs to installed program details ends up stored in the registry as keys and values, organized in a tree structure kind of like nested folders. Regedit is the built in GUI for browsing that structure directly, you navigate down through hives like HKEY_LOCAL_MACHINE or HKEY_CURRENT_USER to find the specific key you're after and change its value from there. 