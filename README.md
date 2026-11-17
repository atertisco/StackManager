# Stack Manager

Stack Manager installs, updates and controls Apache and PHP on Windows.

It prepares each update next to your live version, checks it, carries your settings over, and shows you every change before anything goes live. Apache runs as a normal Windows service. The app is not necessary to keep the server running.

Website: [stackmanager.synsoft.eu](https://stackmanager.synsoft.eu/)

## Download

Download the latest `StackManager-Setup-<version>.exe` from the [Releases](../../releases/latest) page.

To check the file, compare its SHA-256 with the value in the release notes:

```powershell
Get-FileHash .\StackManager-Setup-1.0.0.exe -Algorithm SHA256
```

## Requirements

- Windows 64-bit.
- Administrator rights, to install or change the Apache service.
- A stock Apache zip and a stock PHP zip (see the next section).

## Get Apache and PHP

Stack Manager installs Apache and PHP from the official zip files. Download them from these pages:

| Product | Download page | Which file to get |
| --- | --- | --- |
| Apache | [apachelounge.com/download](https://www.apachelounge.com/download/) | The newest Apache Win64 zip |
| PHP | [php.net/downloads.php?os=windows](https://www.php.net/downloads.php?os=windows) | The newest PHP zip, **x64 Thread Safe** |

> **Note:** For PHP, always select **x64 Thread Safe**. Do not select Non Thread Safe. Apache loads PHP as a module, and this needs the Thread Safe build.

Do not unpack the zip files. Select the zip file itself in the app. Check the download pages again for security releases, and update when a new version is available.

Both products need the Microsoft Visual C++ Redistributable. Each download page tells you which version to install.

## Install Stack Manager

1. Run the installer.
2. Select an install folder that you can write to, for example `C:\StackManager`. Do not use Program Files or the Windows folder.
3. Start Stack Manager.

The install folder is your stack folder. The app keeps Apache, PHP, logs, backups and settings in it.

| Folder | Content |
| --- | --- |
| `bin\apache\apache2.4.x` | The installed Apache version |
| `bin\php\php8.x.x` | The installed PHP version |
| `logs` | Apache error log and access log |
| `backup` | Backup copies made by the app |

## First start

On a new stack, the Dashboard shows a **Get started** card with three steps: install Apache, install PHP, and (optional) import from WAMP. A step is available when the step before it is done.

### Install Apache and the service

1. On the Dashboard, select **Install Apache**.
2. Select the Apache zip file.
3. Select **Prepare**. The app unpacks Apache, sets the log paths and `ServerName localhost`, and runs the Apache check. The Operations panel shows each step.
4. Read the Review panel. All checks must pass.
5. Select **Apply**. The app installs the Windows service `Apache`.

Installing the service needs administrator rights. If the app does not run as administrator, Apply shows a message and keeps the prepared files. You have two options:

- Restart the app as administrator, then select **Apply** again.
- Select **Keep files, install service later**. Then select **Install service** on the Dashboard when the app runs as administrator.

Select **Discard** to delete the prepared files and start again.

### Install PHP

You can install PHP when the Apache files are on disk. The Apache service is not necessary.

1. On the Dashboard, select **Install PHP**.
2. Select the PHP zip file (**x64 Thread Safe**).
3. Select **Prepare**. The app unpacks PHP, creates `php.ini` from `php.ini-production`, and sets `extension_dir` to the `ext` folder of this PHP version.
4. Read the Review panel, then select **Apply**. The app links PHP to Apache.
5. Start or restart Apache on the Dashboard.

The app does not change the Windows PATH when you install PHP for the first time. If the command `php` starts an old PHP, for example the PHP of WAMP, change the PATH yourself.

### Import from WAMP (optional)

Do this last, when Apache and PHP are installed. The import copies your own settings from a WAMP install into the new stack. It does not change the WAMP files.

1. On the Dashboard, select **Import from WAMP**.
2. Select the WAMP folder, for example `G:\WAMP`.
3. Select **Prepare**. The app compares the WAMP files with the stock files and keeps only your changes.
4. Read the Changes, Checks and Review lists. Check each Review item.
5. Select **Apply**, then restart Apache.

The app imports:

- Apache settings such as `ServerName`, `Listen`, `DirectoryIndex`, timeouts and log level.
- Modules and includes, only when the file exists in the new Apache.
- The root `<Directory />` rules `AllowOverride` and `Require`.
- The virtual hosts file, when `httpd.conf` includes it.
- PHP settings from `php.ini`, and extensions that exist in the new PHP.

The app does not import settings that point into the WAMP folder, whole configuration blocks, aliases, or includes outside `conf\`. It does not change the service or the PATH.

Before the app writes a file, it saves a copy as `<file>.before-import`. The first copy is kept. If the new files fail the check, the app restores the old files.

## Daily use

The Dashboard shows the state of the service and the linked PHP.

| Button | What it does |
| --- | --- |
| Start | Starts the service. The app checks the ports first and shows conflicts. |
| Start anyway | Starts the service and ignores port conflicts. |
| Stop | Stops the service. |
| Restart | Stops and starts the service. |
| Reload | Applies a changed configuration without a full stop. |
| Check ports | Shows which program uses the Apache ports. |
| Run checks | Runs the Apache and PHP configuration checks. |

Run the checks after you change a configuration file by hand.

## Update Apache or PHP

Update when Apache Lounge or php.net publishes a security release.

1. Open the **Update** screen and select the **Apache** or **PHP** tab.
2. Select the new zip file and select **Prepare**.
3. The app unpacks the new version next to the old one and moves your settings into the new files. It compares the old stock files, the new stock files and your live files.
4. Read the Changes, Checks and Review lists. Check each Review item.
5. Select **Apply** to switch to the new version.

Select **Discard** to delete the new files. The current version stays unchanged.

## Remove the service and work without the app

**Remove the service.** On the Dashboard, select **Remove service**. The app asks you to confirm, stops the service and deletes it. The Apache and PHP files stay in the stack folder. Administrator rights are necessary.

**Work without the app.** The app is not necessary to run the stack. All files are standard Apache and PHP files:

- Start or stop the service with `net start Apache` and `net stop Apache`, or with `services.msc`.
- Check the configuration with `bin\apache\apache2.4.x\bin\httpd.exe -t`.
- Edit `conf\httpd.conf` in the Apache folder and `php.ini` in the PHP folder.
- Read the logs in the `logs` folder of the stack.

**Uninstall the app.** The uninstaller removes only the app files. It does not change your Apache and PHP files, logs, settings or the Apache service. Delete the stack folder by hand when you do not need it.

## Troubleshooting

| Problem | Cause | Action |
| --- | --- | --- |
| `Access is denied` or `AH00369` when you select Apply | The app does not run as administrator. | Restart the app as administrator. Or select **Keep files, install service later**. |
| **Prepare** is not available | No zip is selected. For PHP, no Apache files exist. For the import, Apache or PHP is not installed. | Select a zip. Install Apache before PHP and the import. |
| Warning `AH00558` about the server name | Apache has no `ServerName`. | The app sets `ServerName localhost`. It is a warning, not an error. |
| The service does not start or times out | A configuration error, or a port in use. | Select **Check ports** and **Run checks**. Read `logs\apache_error.log`. |
| Two Apache services, for example `Apache` and `Apache2.4` | The WAMP service is still installed. | Stop the WAMP service. Both services use the same ports. |
| `Unable to load dynamic library` | `extension_dir` in `php.ini` points to a wrong folder. | Set `extension_dir` to the `ext` folder of the installed PHP. |
| The command `php` starts an old PHP | The PATH points to another PHP. | Change the PATH to the PHP folder in `bin\php`. |
| A virtual host does not work after the import | Its folder is on a drive that is not available. | Connect the drive, or change the path in the virtual hosts file. |

## About

Stack Manager is made by [Synsoft Solutions](https://www.synsoft.eu/).

&copy; 2026 Synsoft Solutions
