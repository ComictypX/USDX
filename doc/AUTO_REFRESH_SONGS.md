# Automatic Song Folder Monitoring

## Overview
UltraStar Deluxe can now automatically detect changes to your song library without requiring a restart. This feature monitors the song directories and updates the song list when new songs are added, existing songs are modified, or songs are deleted.

## Configuration

The auto-refresh feature can be enabled/disabled in the game settings:

1. Go to **Options** → **Advanced Settings**
2. Find the **AutoRefreshSongs** option
3. Set it to **On** to enable automatic monitoring (default is **Off**)

The setting is saved in `config.ini` under the `[Advanced]` section:
```ini
[Advanced]
AutoRefreshSongs=On
```

## How It Works

When enabled, the system:
- Checks for changes every **5 seconds**
- Scans all configured song directories
- Detects new `.txt` song files and adds them to the library
- Removes songs that have been deleted
- Updates the UI automatically (covers, thumbnails, song list)

## Performance

The incremental update is efficient:
- Only scans for actual changes, not a full reload
- New songs are analyzed and added individually
- Deleted songs are removed from memory
- UI updates are minimal and non-disruptive
- Designed to handle libraries with 1000+ songs

## Usage Scenarios

### Adding New Songs
1. Copy new song folders to your songs directory
2. Within 5 seconds, the new songs will appear in the song list
3. No restart required

### Deleting Songs
1. Delete song folders or `.txt` files
2. Within 5 seconds, those songs will be removed from the list
3. No restart required

### Modifying Songs
Currently, the system detects when song files exist or are deleted. To see changes to existing song files (like updated metadata), you will need to trigger a full reload by restarting the game or manually reloading the song list.

## Thread Safety

The monitoring runs in a background thread and is designed to be safe:
- Uses the existing `Processing` flag to prevent concurrent modifications
- Only updates when the system is idle
- Does not interfere with normal gameplay or song selection

## Backward Compatibility

- Default setting is **Off** to maintain existing behavior
- No changes to existing code paths when disabled
- Config file remains compatible with older versions

## Technical Details

### Implementation
- Based on the proven timestamp pattern used for playlist monitoring
- Implements efficient incremental updates rather than full reloads
- Integrates into the existing `TSongs.Execute()` thread loop
- Uses `FindFilesByExtension()` for consistent file scanning

**Platform Limitations:**
- **macOS Debug Builds**: The feature does not work when `USE_PSEUDO_THREAD` is defined (macOS debug mode)
- **Windows/Linux**: Full support
- **macOS Release**: Full support

The monitoring requires a real thread loop, which is not available in pseudo-thread mode used for macOS debugging.

### Code References
- `src/base/UIni.pas` - Configuration option
- `src/base/USongs.pas` - Monitoring logic
  - `CheckForChanges()` - Detects if scan is needed
  - `IncrementalUpdate()` - Performs efficient updates

## Troubleshooting

### Songs not appearing automatically
- Check that AutoRefreshSongs is set to **On** in Advanced settings
- Verify the song files have `.txt` extension
- Ensure files are in a configured song directory
- Wait at least 5 seconds for the next scan cycle

### Feature not working on macOS (Debug builds)
**Important Limitation:** The auto-refresh feature currently does **not work** on macOS when running debug builds. This is due to the use of pseudo-threads on macOS debug mode, which don't support the background monitoring loop.

**Workaround:** 
- Use a release build on macOS for auto-refresh functionality
- Or restart the game to see song library changes

This limitation affects only macOS debug builds; Windows, Linux, and macOS release builds work as expected.

### Performance issues
- If you have a very large song library (5000+ songs), consider keeping auto-refresh **Off**
- The system is optimized but frequent scans on slow storage (network drives) may cause delays

## Future Enhancements

Potential improvements for future versions:
- Configurable scan interval (5/10/30 seconds)
- Detection of modified song files (not just new/deleted)
- Manual trigger option (e.g., F5 key in song screen)
- Visual indicator during scan
- Optional notification when new songs are found
- Native file watching APIs (inotify on Linux, ReadDirectoryChangesW on Windows, FSEvents on macOS)
