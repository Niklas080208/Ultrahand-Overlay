# Update Checker Package Example

This example demonstrates how to implement automatic update checking for your Ultrahand package.

## Features

- **Version Checking**: Downloads a remote version file and compares it with the local version
- **Update Notifications**: Shows a notification displaying both remote and local versions
- **Update Installation**: Provides a command to download and install updates

## How It Works

Since Ultrahand packages don't support traditional conditional logic (if/else), this implementation:
1. Downloads the remote version file
2. Compares it with the local version
3. Shows a notification with **both** versions displayed
4. The user can visually compare and decide if an update is needed

For automatic update detection, you can modify the notification message to be more explicit, or implement a flag-file based system (see Advanced section below).

## Setup Instructions

### 1. Host Your Version File

Create a `VERSION.ini` file on your server (GitHub raw URL, web server, etc.) with the following format:

```ini
[Release Info]
latest_version=1.0.1
```

### 2. Configure the Package

Edit `package.ini` and modify these values:

- **Remote Version URL**: Change `https://raw.githubusercontent.com/yourusername/yourrepo/main/VERSION.ini` to your actual URL
- **Package Name**: Replace `Update Checker` with your package name in file paths
- **Update Download URL**: Modify the download URL in the `[*Update Package]` section
- **INI Section/Key**: If your version file uses different section/key names, update:
  - `Release Info` (section name)
  - `latest_version` (key name)

### 3. Create Local Version File

The package will automatically create `/switch/.packages/Update Checker/local_version.ini` on first run with your current version.

### 4. Usage

1. **Check for Updates**: Run the `[*Check for Updates]` command
   - Downloads the remote version file
   - Compares with local version
   - Shows notification if update is available

2. **Update Package**: Run the `[*Update Package]` command
   - Downloads the latest release
   - Installs it to your package directory
   - Updates the local version file

## Advanced: Boot-Time Update Check

To check for updates automatically on boot, create a `boot_package.ini` file in your package directory:

```ini
[boot]
; Check for updates on boot
try:
delete /config/ultrahand/downloads/remote_version.ini
download https://raw.githubusercontent.com/yourusername/yourrepo/main/VERSION.ini /config/ultrahand/downloads/remote_version.ini
ini_file /config/ultrahand/downloads/remote_version.ini
set-ini-val /config/ultrahand/downloads/version_check.ini 'Version Check' remote_version {ini_file(Release Info,latest_version)}
try:
set-ini-val /switch/.packages/Update Checker/local_version.ini 'Release Info' latest_version 1.0.0
ini_file /switch/.packages/Update Checker/local_version.ini
set-ini-val /config/ultrahand/downloads/version_check.ini 'Version Check' local_version {ini_file(Release Info,latest_version)}
ini_file /config/ultrahand/downloads/version_check.ini
notify-now 'Update Check: Remote: {ini_file(Version Check,remote_version)} | Local: {ini_file(Version Check,local_version)}' 22
delete /config/ultrahand/downloads/remote_version.ini
delete /config/ultrahand/downloads/version_check.ini
```

Place this file at: `/switch/.packages/Update Checker/boot_package.ini`

## Advanced: Flag-Based Update Detection

For a more automated approach, you can create a flag file only when an update is available. However, this requires manual version comparison logic. A simpler alternative is to always show the notification with both versions, allowing users to see at a glance if they need to update.

## Version Comparison Notes

The example uses simple string comparison. For semantic versioning (e.g., "1.0.0" vs "1.0.1"), this works well because:
- String comparison works correctly for versions like "1.0.0" < "1.0.1" < "1.1.0" < "2.0.0"
- For more complex version formats, you may need to implement custom comparison logic

## Alternative: Using JSON for Version Info

If you prefer JSON format, you can modify the package to use `json_file` instead:

```ini
[*Check for Updates]
try:
delete /config/ultrahand/downloads/remote_version.json
download https://raw.githubusercontent.com/yourusername/yourrepo/main/version.json /config/ultrahand/downloads/remote_version.json
json_file /config/ultrahand/downloads/remote_version.json
set-ini-val /config/ultrahand/downloads/version_check.ini 'Version Check' remote_version {json_file(version)}
; ... rest of the logic
```

## Troubleshooting

- **No notification shown**: Check that notifications are enabled in Ultrahand settings
- **Download fails**: Verify the URL is accessible and the file exists
- **Version comparison not working**: Ensure both version files use the same format

## License

This example is provided as-is for educational purposes. Modify as needed for your package.

