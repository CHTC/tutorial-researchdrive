# Tutorial: Using ResearchDrive with CHTC

Learn how to use the computing power of CHTC with the data you keep in ResearchDrive.

## Overview

* [What is ResearchDrive?](#what-is-researchdrive)
* [Manual transfers](#manual-transfers)
* [Automated transfers](#automated-transfers)
* [UWDF: Beyond ResearchDrive](#uwdf-beyond-researchdrive)

## What is ResearchDrive?

ResearchDrive ([it.wisc.edu/services/researchdrive/](https://it.wisc.edu/services/researchdrive/)) is "a secure and permanent place for keeping data" for research groups at UW Madison.
Storage and access is managed by the PI of the research group, and each eligible research group gets 25TB of storage for free.

CHTC users have a few use cases for ResearchDrive:

* **Long term backup**: CHTC does not back up user data - ResearchDrive is the perfect resource for this!
* **Storing large inputs/outputs**: CHTC has a finite amount of space for user data - ResearchDrive can be used as a supplement.
* **Sharing data with collaborators**: ResearchDrive data can be shared with users outside of UW, without needing a CHTC account.

### Restricted ResearchDrive

ResearchDrive has a separate, dedicated system for handling protected data (e.g., personal identifying information).
**CHTC cannot (and should not!) access this Restricted ResearchDrive.**

CHTC systems are not rated for handling protected data.
You must not try to circumvent this, as you may break the law(s) protecting the data!!

There are other resources on campus rated for computing on protected data, if that is something you need.

> [!TIP]
> Not sure if your data is "protected"? In general, if the data is publically accessible (without requiring login/authentication) from a reputable source, then it is fine to use on CHTC.
> Feel free to check with the facilitation team for assistance!

### Getting access to ResearchDrive

**If your research group already has a ResearchDrive**, then you will need to work with your group members to get access.
The PI of the research group (or their designate) control the access to their ResearchDrive.

**If your group does not yet have a ResearchDrive**, the PI or their designate needs to complete the [Request Account](https://it.wisc.edu/services/researchdrive/) form.

> [!NOTE]
> CHTC does not manage ResearchDrive!
> If you have questions about the account process or getting access, you should contact the ResearchDrive team at [researchdrive@wisc.edu][mailto:researchdrive@wisc.edu].

### Accessing ResearchDrive

There are a few ways of accessing data in ResearchDrive, but the most common is "mounting" the drive to your computer as a network drive.
Once mounted, ResearchDrive appears as just another folder on your computer that you can interact with.

![Windows Explorer view of a mounted ResearchDrive folder]()

Setting this up is not the focus of our training; if you are interested in this, see their guides [for Windows](https://kb.wisc.edu/researchdata/96187), [for MacOS](https://kb.wisc.edu/researchdata/96187), or [for Linux](https://kb.wisc.edu/researchdata/95148).

> [!CAUTION]
> Do not use the Linux instructions to connect to ResearchDrive from a CHTC server!
> The exception is for the methods we discuss in this training.

## Manual transfers

You can manually transfer data to/from ResearchDrive and CHTC via the command line.
With this method, you are in full control of the data movement.

The following approach is also described in our guide [Transfer Files Between CHTC and ResearchDrive](https://chtc.cs.wisc.edu/uw-research-computing/transfer-data-researchdrive)

### Assumptions

* You have access to a ResearchDrive and know its address
* Your NetID has permission to access the desired data in the ResearchDrive
* You have a CHTC account
* Your CHTC account has permission to access the desired data on the CHTC server

### How it works

1. You login to the CHTC server
2. You use a file transfer client (`smbclient`) to login to your ResearchDrive
3. You initiate transfers to/from CHTC
4. Wait for transfers to complete

> [!IMPORTANT]
> You must remain connected to the CHTC server for the full duration of the transfer!
> While the data is transferred directly between CHTC and ResearchDrive servers, your active login is required to monitor the transfer.

### Logging in to CHTC

To transfer data to/from a CHTC server, you first need to be logged into the correct server.

* **HTC /home** - To transfer data to/from your `/home` directory on the HTC system, you need to login to your access point, typically `ap2001.chtc.wisc.edu` or `ap2002.chtc.wisc.edu`.
* **HTC /staging** - To transfer data to/from your `/staging` (or `/projects`) directory, you need to login to transfer server at `transfer.chtc.wisc.edu`. (Remember to `cd /staging/yourNetID` before transferring data!)
* **HPC** - To transfer data to/from your directories on the HPC system, you need to login to as normal to `spark-login.chtc.wisc.edu`.

### Connecting to ResearchDrive

Next, move into the directory on the CHTC server you want to work with.
For example, let's say that you have a `experiments` directoy in `/staging`:

```bash
cd /staging/yourNetID/experiments
```

When ready, run this command:

```bash
smbclient -k //research.drive.wisc.edu/<ResearchDrive_Name>
```

Here, you will need to replace `<ResearchDrive_Name` with the name assigned to your group's ResearchDrive, which typically involves the PI's name or NetID.

For example, if you are trying to access the ResearchDrive of Prof. Bucky Badger, your command might look like this:

```bash
smbclient -k //research.drive.wisc.edu/bbadger
```

> [!CAUTION]
> **If the address for your ResearchDrive is `restricted.drive.wisc.edu`**, then you are trying to access a Restricted ResearchDrive, which will fail!
> See the [Restricted ResearchDrive section](#restricted-researchdrive) above.

> [!TIP]
> **Not sure what your ResearchDrive address is?** You can run this command to check which ResearchDrives you have access to:
>
> ```bash
> smbclient -L //research.drive.wisc.edu/
> ```
>
> The `<ResearchDrive_Name>` values that you can use will be listed under the `Sharename` column in the output.
> If the list is empty, you don't have access to ResearchDrive (or you have a Restricted ResearchDrive).

If the command is successful, you should see this message:

```bash
Try "help" to get a list of possible commands.
smb: \>
```

You are now in an interactive prompt for using the `smbclient` to transfer data to/from ResearchDrive.
This works sort of like a regular command line, but with fewer possible commands that don't always work the way you expect.

> [!TIP]
> You can ignore the `WARNING: The option -k|--kerberos is deprecated!` message for now.
> The "correct" command to avoid this warning is to do
>
> ```bash
> smbclient --use-kerberos=desired //research.drive.wisc.edu/<ResearchDrive_Name>
> ```
>
> You don't need to know what "kerberos" is, other than that it enables you to "re-use" your authentication from when you logged into the CHTC server using your NetID.
> 
> Alternatively, you can use
>
> ```bash
> smbclient -U yourNetID //research.drive.wisc.edu/<ResearchDrive_Name>
> ```
>
> in which case you'll need to enter your NetID password when prompted.

You can see the full list of commands by running `help`, and see the help text for specific commands using `help commandName`.
Note that not all commands are enabled/available in the CHTC/ResearchDrive setup.

You can exit the `smbclient` prompt using most of the methods you are used to: `q`, `quit`, `exit`, `Ctrl`+`C` shortcut, and `Ctrl`+`D` shortcut.

> [!CAUTION]
> Some folks use `Ctrl`+`C` to their command prompt when they've made a mistake.
> Doing so in the `smbclient` prompt will cause it to exit!

### Listing your data in ResearchDrive

Using the `smbclient` command line, you find your data in ResearchDrive through combinations of `ls` and `cd` commands.

Running `ls` after starting the `smbclient` will show you the top level contents of your ResearchDrive.
Note how the output looks different from that of the Unix `ls` command.

If you want to see the contents of a directory in your ResearchDrive, you have to first `cd` into that directory.

> [!TIP]
> If you tab-autocomplete a directory name, you will see a backslash (`\`) appear at the end of the name, when usually in Unix you see a forward slash (`/`).
> Either one is acceptable in this case.

### Downloading a file from ResearchDrive to CHTC

To download a file from ResearchDrive, run

```bash
get <file>
```

where `<file>` is the name of - or path to - a file in your ResearchDrive.

Optionally, you can change how the file is named in your CHTC directory with

```bash
get <file> <newname>
```

By default, the file will be returned to the directory **where you ran the `smbclient` command**.
You can have the file returned to a different location by using a relative or absolute path:

```bash
get <file> /home/yourNetID/<newname>
```

> [!WARNING]
> Make sure you use the forward slash (`/`) in this case.
> If you use a backslash (`\`) in the path, you'll create a file in the initial directory with a backslash in its name!

> [!TIP]
> Tab-autocomplete should work as expected for specifying the file in ResearchDrive or the location in CHTC to transfer it to!

### Downloading many files from ResearchDrive to CHTC

To download more than one file at a time, you cannot just use the `get` command.

You need to use the `mget` command.

With the `mget` command, you can specify multiple names at a time, either by listing them one at a time or by using a "glob".

For example, to download all the files from your current ResearchDrive directory that end with `.txt`, you would use

```bash
mget *.txt
```

**But** the default behavior is to *prompt you to confirm every single transfer* (!).

To disable this prompt, run the command

```bash
prompt
```

> [!TIP]
> The `prompt` command is an invisible "toggle" - it won't tell you whether prompting is on or off!
> The first time you run the command in the `smbclient` session, you will toggle the state from "on" (the default) to "off".
> The second time, you'll toggle the state from "off" to "on", and so on and so forth.
> If you forget which state the toggle is in, just exit and restart the `smbclient` command line to reset the state to the default of "on".

Now when you run the `mget` command, it will just do the transfers instead of prompting you to confirm each one.

### Uploading a file from CHTC to ResearchDrive

You can use the `put` command to transfer files from CHTC to ResearchDrive.

When you are using the `smbclient`, you can't `ls` your files & directories on CHTC.
However, you can use the tab-autocomplete functionality to display the possible autofill options.

If you launched the `smbclient` in the same directory as the file, and you remember the filename, you can just run

```bash
put <file>
```

You can also specify a different name to save the file in ResearchDrive

```bash
put <file> <newname>
```

You can also specify paths - remember to use forward slashes for specifying locations on CHTC.

### Uploading many files from CHTC to ResearchDrive

We complete the square with the `mput` command.

* `put` is for uploading one file, `mput` is for many files
* You can use globs (e.g., `*`) to transfer files based on a pattern
* The `prompt` toggle affects the `mput` command as well for user confirmation

For example, if you have a bunch of `.csv` files in your CHTC directory you want to upload to ResearchDrive,
you could use

```bash
mput *.csv
```

## Automated transfers

**What you can connect to**
**What CHTC can see/do**

## UWDF: Beyond ResearchDrive

While ResearchDrive is the focus of this training, the technology that enables the automated transfer can be used for transfering data from other sources.
We call the system "UWDF" and it uses the software [Pelican Platform](https://pelicanplatform.org) to integrate with other data storage systems. 

