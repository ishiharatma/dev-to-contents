---
title: Efficiently Managing AWS CLI Profiles Across Multiple Projects with PowerShell
published: false
description: A practical PowerShell solution for managing AWS CLI credentials and config files across multiple projects
tags: 'aws, powershell, devops, automation'
cover_image: ''
canonical_url: null
id: 2340638
---

## ☘️ Introduction

If you work with multiple AWS projects, you've likely encountered the challenge of managing your AWS CLI configuration files (`~/.aws/config` and `~/.aws/credentials`). These files tend to grow unwieldy over time. While AWS IAM Identity Center can manage credentials for accounts within AWS Organizations, there are scenarios where you need multiple login credentials for individual accounts or multi-account setups using jump accounts:

- Situations where AWS IAM Identity Center cannot be used due to organizational constraints
- Managing multiple projects and their associated customer accounts
- During transition periods when multiple authentication methods temporarily coexist

In these cases, adding new profiles and removing outdated ones becomes a complex task prone to errors.

While juggling several AWS projects, I found myself frustrated by this exact problem. So I created a PowerShell script that allows me to manage configuration files separately for each project and automatically merge them when needed.

You can find the complete source code in [this GitHub repository](https://github.com/ishiharatma/aws-cli-config-manager).

![overview](./assets/aws-cli-config-manager.drawio.svg)

### Alternative Approaches

Another way to manage AWS CLI credentials is by using the [AWS_SHARED_CREDENTIALS_FILE](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html#envvars-list) environment variable. This approach involves physically switching between different credential files:

1. Create multiple credentials files.

    ```text
    ~/.aws/credentials.dev
    ~/.aws/credentials.prod
    ~/.aws/credentials.test
    ```

2. Set up authentication information in each file.

    ```text
    [default]
    aws_access_key_id = {your_access_key}
    aws_secret_access_key = {your_secret_key}
    ```

3. Specify the file to use via environment variable.

    ```text
    export AWS_SHARED_CREDENTIALS_FILE=~/.aws/credentials.dev
    ```

While this method works, it requires resetting environment variables each time you need to switch contexts, making it cumbersome when working with multiple projects simultaneously.
The PowerShell script proposed in this article offers a more efficient solution by managing all project settings in a single credentials file, making it easy to add or remove project configurations as needed. Switching between projects simply requires specifying the profile when using AWS CLI, resulting in a more streamlined workflow.

## 📌 Challenges and Solutions

### Challenges

- AWS CLI configuration files become unwieldy as they grow
- Manual editing is error-prone and time-consuming
- Managing settings on a per-project basis is difficult

### Solutions

This script addresses these issues with the following approach:

- Manage settings in separate files for each project
- Control ordering with filename conventions
- Automate merging and backup processes
- Detect differences to perform efficient updates

## 📒 Solution Overview

The script works with the following simple process:

1. Place project-specific files in `credentials` and `configs` directories
   - Using the naming format: `credentials.yyyy-mmdd.name`
   - Using the naming format: `config.yyyy-mmdd.name`
2. Sort these files alphabetically and merge them
3. Compare the merged file with the actual AWS CLI configuration files
4. If no differences are found, end the process
5. If differences exist, back up existing files and update them with the merged version

## 👀 How to Use It

### Setup

1. Create two folders in your working directory: `credentials` and `configs`
2. Save each project's credentials and settings using the following naming convention:
   - Credentials: `credentials.yyyy-mmdd.project-name`
   - Config: `config.yyyy-mmdd.project-name`

For example:

```text
./
├── credentials/
│   ├── credentials.2024-0301.project-a
│   ├── credentials.2024-0310.project-b
│   └── credentials.2024-0315.project-c
└── configs/
    ├── config.2024-0301.project-a
    ├── config.2024-0310.project-b
    └── config.2024-0315.project-c
```

### Running the Script

Place the script in your working directory and run it with:

```powershell
.\aws-cli-config-manager.ps1
```

To run in debug mode (which shows what would happen without making changes):

```powershell
.\aws-cli-config-manager.ps1 -Debug
```

### Execution Examples

Output when running in debug mode:

```powershell
PS> .\aws-cli-config-manager.ps1 -Debug
[DEBUG] Running in debug mode
[DEBUG] Current working directory: C:\Users\username\aws-settings
[DEBUG] AWS credentials path: C:\Users\username\.aws\credentials
[DEBUG] AWS config path: C:\Users\username\.aws\config
[DEBUG] Credentials directory: C:\Users\username\aws-settings\credentials
[DEBUG] Backup path: C:\Users\username\.aws\credentials.bak
[DEBUG] Found 3 files (pattern: credentials.*):
[DEBUG]   - C:\Users\username\aws-settings\credentials\credentials.2024-0301.project-a
[DEBUG]   - C:\Users\username\aws-settings\credentials\credentials.2024-0310.project-b
[DEBUG]   - C:\Users\username\aws-settings\credentials\credentials.2024-0315.project-c
...
[DEBUG] Completed debug mode run - no actual changes were made
```

Output when running in normal mode:

```powershell
PS> .\aws-cli-config-manager.ps1
Backed up existing file to: C:\Users\username\.aws\credentials.bak
Successfully updated file: C:\Users\username\.aws\credentials
Backed up existing file to: C:\Users\username\.aws\config.bak
Successfully updated file: C:\Users\username\.aws\config
Process completed: 2 file(s) updated
```

## 🔧 Script Details

The script includes several key features:

1. **File Merging**: Sorts and merges files from the specified directories
2. **Line Ending Preservation**: Uses binary mode processing to maintain line endings
3. **Difference Detection**: Compares files at the byte level to detect changes
4. **Automatic Backup**: Creates backups of existing files before updating
5. **Debug Mode**: Allows you to preview changes without applying them

### Core Function: `Update-AwsFile`

The heart of the script is this function:

```powershell
function Update-AwsFile {
    param (
        [string]$SourceDir,        # Source file directory
        [string]$FilePattern,      # File pattern (credentials.* or config.*)
        [string]$FileRegex,        # Filename regex pattern
        [string]$TmpFileName,      # Temporary filename
        [string]$TargetFile,       # Target file path (.aws/credentials or .aws/config)
        [string]$BackupFile        # Backup filename
    )

    # Processing logic...
}
```

This function handles:

1. Finding and sorting matching files
2. Merging them into a temporary file
3. Checking for differences with existing files
4. Performing backups and updates as needed

### Difference Detection Logic

The script uses binary comparison for accurate difference detection:

```powershell
# Compare files in binary mode
$existingBytes = [System.IO.File]::ReadAllBytes($TargetFile)
$newBytes = [System.IO.File]::ReadAllBytes($tmpFilePath)

# If file sizes differ, mark as different
if ($existingBytes.Length -ne $newBytes.Length) {
    $isDifferent = $true
} else {
    # Compare byte by byte
    for ($i = 0; $i -lt $existingBytes.Length; $i++) {
        if ($existingBytes[$i] -ne $newBytes[$i]) {
            $isDifferent = $true
            break
        }
    }
}
```

## 🌟 Benefits of This Approach

1. **Project-Based Management**: Each project's settings are in separate, easy-to-manage files
2. **Chronological Control**: Date-based filenames enable easy timeline management
3. **Safe Updates**: Automatic backups protect your configuration
4. **Efficient Processing**: Updates only happen when differences are detected
5. **Risk Mitigation**: Debug mode lets you verify changes before applying them

## 💡 Pro Tips for Daily Use

### Filename Tricks

The date-based naming scheme allows you to control the sort order:

```text
credentials.2024-0301.project-a  # Older project
credentials.2024-0310.project-b  # Middle project
credentials.2024-0315.project-c  # Newer project
```

Want certain profiles to always appear first or last? Adjust the date portion:

```text
credentials.0000-0000.always-first  # Always appears first
credentials.9999-9999.always-last   # Always appears last
```

### ⚠ Important Considerations

- If identical profile names exist in multiple files, you'll get an `Unable to parse config file` error when executing AWS CLI commands, so avoid duplicate profile names
- Use debug mode to verify that the merged result matches your expectations before applying changes

## 📖 Conclusion

This PowerShell script has significantly simplified my AWS CLI configuration management across multiple projects. Since implementing this solution, I've experienced several tangible benefits:

- **Reduced Maintenance Time**: Configuration file maintenance time reduced
- **Error Prevention**: Eliminated errors caused by manual editing
- **Flexible Management**: Adding or removing project settings became much easier

The script is also highly extensible. You could enhance it to selectively merge files based on specific criteria, set up scheduled updates, or integrate it with your DevOps pipelines.

I hope this tool helps you manage your AWS CLI configuration more efficiently.

Happy cloud computing!

Follow me for more tips :)
