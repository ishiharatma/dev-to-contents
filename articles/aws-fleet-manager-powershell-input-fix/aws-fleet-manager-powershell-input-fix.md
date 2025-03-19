---
title: Resolving PowerShell Text Input Issues on Windows Server 2022 EC2 via Fleet Manager
published: false
description: A quick solution for fixing keyboard input problems in PowerShell when connecting to Windows Server 2022 EC2 instances through Fleet Manager
tags: 'aws, powershell, ec2, troubleshooting'
cover_image: ./assets/cover.svg
canonical_url: null
organization_id: 2794
id: 2342672
---

## 🚨 The Problem

If you've connected to a Windows Server 2022 EC2 instance via Fleet Manager, you might have encountered this frustrating issue in the PowerShell console:

- Direct keyboard text input doesn't work
- Only the Enter key functions properly
- You can copy & paste from external sources

This situation severely impacts productivity since you need to pre-compose all commands in a text editor before pasting them into PowerShell. Let me walk you through the cause and solution to this problem.

## 🔍 Root Cause Identified

After some investigation, I identified that the default installed version of `PSReadline 2.0.0` is causing this keyboard input issue.

For reference, there's a GitHub issue discussing a similar problem: [PSReadline 2.1.0 doesn't work in the windows virtual desktop web interface #2725](https://github.com/PowerShell/PSReadLine/issues/2725)

## 💻 Environment Details

**AMI Used:**
Windows_Server-2022-English-Full-Base-2025.02.13

## 🔎 How to Verify the Issue

To confirm you're experiencing the same problem, copy and paste this command to check your PSReadline version:

```powershell
Get-Module
```

You should see output similar to this:

```powershell
ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PSReadLineKeyHandler, Set-PSReadLineKeyH...
```

If your PSReadline version is 2.0.0, you're definitely facing this issue.

## ✅ The Solution

This simple procedure will install the latest PSReadline version (2.3.6 as of February 2025) and resolve the keyboard input problem:

1. Open Windows PowerShell from the Start menu
2. Verify that text input is not working
3. Install PSReadLine version 2.2.2 or higher which contains the fix by copying and pasting this command:
   ```powershell
   Install-Module -Name PSReadLine -Repository PSGallery -MinimumVersion 2.2.2 -Force
   ```
4. Close the current PowerShell console and open a new one
5. Check that the new version is installed with:
    ```powershell
    Get-Module
    ```
    
    If the version shows 2.3.6 (or newer), the upgrade was successful:
    ```powershell
    ModuleType Version    Name                                ExportedCommands
    ---------- -------    ----                                ----------------
    Script     2.3.6      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...
    ```
6. Finally, confirm that keyboard text input now works properly

## 💡 Why This Happens

The PSReadline module handles keyboard input in PowerShell, and version 2.0.0 has compatibility issues with remote console interfaces like Fleet Manager. Upgrading to version 2.2.2 or higher includes fixes for these web interface connectivity problems.

## 📝 Final Thoughts

This simple fix saves you from the frustration of not being able to type commands directly into PowerShell when managing your Windows Server EC2 instances through Fleet Manager. No more copying and pasting every single command!

If you encounter any other PowerShell issues with AWS environments, feel free to share in the comments.

Happy cloud computing!

Follow me for more AWS and PowerShell tips :)
