---
title: Windows.System.PowerShell
hidden: true
tags: [Client Artifact]
---

This artifact allows running arbitrary commands through the system
powershell.

Since Velociraptor typically runs as system, the commands will also
run as System.

This is a very powerful artifact since it allows for arbitrary
command execution on the endpoints. Therefore this artifact requires
elevated permissions (specifically the `EXECVE`
permission). Typically it is only available with the `administrator`
role.

Note that in addition to running PowerShell cmdlets and scripts, the
Windows.System.PowerShell artifact can also be used to launch
Windows command-line executables with their parameters. This can be
difficult to achieve with the Windows.System.CmdShell artifact due
to complications with spaces in paths and other special character
issues. This PowerShell artifact is able to avoid most of these
problems by encoding the command in Base64.

As an example, the following command initiates a Windows Defender AV
quick-scan from the default location, which includes a path with
spaces in it:

```
  & 'C:\Program Files\Windows Defender\MpCmdRun.exe' -Scan -ScanType 1
```


```yaml
name: Windows.System.PowerShell
description: |
  This artifact allows running arbitrary commands through the system
  powershell.

  Since Velociraptor typically runs as system, the commands will also
  run as System.

  This is a very powerful artifact since it allows for arbitrary
  command execution on the endpoints. Therefore this artifact requires
  elevated permissions (specifically the `EXECVE`
  permission). Typically it is only available with the `administrator`
  role.

  Note that in addition to running PowerShell cmdlets and scripts, the
  Windows.System.PowerShell artifact can also be used to launch
  Windows command-line executables with their parameters. This can be
  difficult to achieve with the Windows.System.CmdShell artifact due
  to complications with spaces in paths and other special character
  issues. This PowerShell artifact is able to avoid most of these
  problems by encoding the command in Base64.

  As an example, the following command initiates a Windows Defender AV
  quick-scan from the default location, which includes a path with
  spaces in it:

  ```
    & 'C:\Program Files\Windows Defender\MpCmdRun.exe' -Scan -ScanType 1
  ```

required_permissions:
  - EXECVE

precondition:
  SELECT OS From info() where OS = 'windows'

parameters:
  - name: Command
    default: "dir C:/"
  - name: PowerShellExe
    default: "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe"

sources:
  - query: |
      SELECT * FROM execve(argv=[PowerShellExe,
        "-ExecutionPolicy", "Unrestricted", "-encodedCommand",
        base64encode(string=utf16_encode(string=Command))
      ])

```