## Windows Update

Windows Update is the Microsoft service that handles security updates, feature updates, and patches for Windows and other Microsoft products like Defender. You can go to it from the Run dialog or cmd with `control /name Microsoft.WindowsUpdate`.

## Windows Security

Windows Security also lives in Settings, split into a few protection areas: Virus & threat protection, Firewall & network protection, App & browser control, and Device security.

### Virus & threat protection

This splits into Current threats and Virus & threat protection settings.

Under Current threats, there's three scan types: 

Quick scan checks the folders threats are commonly found in
Full scan checks every file and running program on the disk 
Custom scan lets you pick specific files or locations to check. 

Threat history shows the last scan, any quarantined threats, and allowed threats.

Virus & threat protection settings is where the actual controls live:

- Real-time protection: actively stops malware from installing or running.
- Cloud-delivered protection: taps into Microsoft's latest threat data in the cloud for faster detection.
- Automatic sample submission: sends suspicious files to Microsoft to help improve detection for everyone.
- Controlled folder access: locks down specific folders so only trusted, approved apps can modify what's in them, toggled on from the Manage controlled folder access button.
- Exclusions: lets you tell the antivirus to skip scanning specific files or folders, mainly to cut down false positives, added through Add or remove exclusions.
- Notifications: controls whether Defender pings you with updates on the device's security health.

### Firewall & network protection

A firewall controls what traffic is and isn't allowed to pass in and out through a device's ports, basically a gatekeeper checking everything trying to get in or out.

Windows Firewall runs three separate profiles:

- Domain: applies when the machine can authenticate to a domain controller.
- Private: a user-assigned profile for home or private networks.
- Public: the default profile, used for public networks like coffee shop or airport Wi-Fi.

Clicking into any one of these profiles gives you two options, turn that profile's firewall on or off, or block all incoming connections for it.

### App & browser control

This is where Microsoft Defender SmartScreen lives, it protects against phishing sites, malware sites and apps, and downloading files that are potentially malicious. It's got three states, Warn, Block, or Off.

Under Check apps and files, SmartScreen flags unrecognized apps and files coming from the web before they run. 

### Device security

Core isolation has Memory Integrity, which stops malicious code from getting inserted into high-security processes by isolating them in a protected area of memory.

TPM (Trusted Platform Module) is a dedicated crypto-processor built into the hardware that handles security related functions, cryptographic operations mainly. It's built to be tamper-resistant, physically hardened so malicious software can't mess with what it's doing.

## BitLocker

BitLocker is Microsoft's built-in drive encryption feature, meant to protect data if a device gets lost, stolen, or decommissioned without being wiped properly. It works best paired with a TPM (version 1.2 or later), the TPM works alongside BitLocker to protect the data and confirm the system hasn't been tampered with while it was offline. 
## Volume Shadow Copy Service (VSS)

VSS creates consistent shadow copies, basically point-in-time snapshots, of data that's being backed up. These snapshots get stored in the System Volume Information folder on whichever drives have protection enabled.

With VSS on (System Protection enabled), you can create a restore point, run a system restore, configure restore settings, or delete existing restore points.