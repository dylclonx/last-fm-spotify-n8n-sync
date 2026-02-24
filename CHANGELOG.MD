# Changelog

All notable changes to this project will be documented in this file.

## [1.1.6] - 2026-02-23

### Fixed
- **Authentication**: Resolved persistent 403 Forbidden errors by re-authenticating Spotify credentials and correctly managing authorized users in the Spotify Developer Dashboard.
- **Node Regressions**: Re-applied fixes for native Spotify nodes (Add/Remove) and Merge node to restore missing parameters (`operation`, `playlistId`, `trackUris`, `mode`) that were lost during the revert.

### Changed
- **Spotify Nodes**: Reverted from direct `HTTP Request` nodes back to native `Spotify` nodes following the resolution of credential-related permission issues.

### Known Issues
- **Rate Limiting**: Identified Spotify API rate-limiting as a potential bottleneck during large sync operations.

## [1.1.5] - 2026-02-23

### Changed
- **Spotify API Strategy**: Migrated all Spotify nodes to direct `HTTP Request` nodes to use Generic OAuth2 credentials. This was a troubleshooting step to bypass 403 Forbidden errors.

## [1.1.4] - 2026-02-23

### Changed
- **Spotify API Strategy**: Replaced the native `Add New Tracks` Spotify node with a direct `HTTP Request` node. This bypasses potential 403 Forbidden errors caused by n8n's built-in node scope handling and ensures more reliable playlist updates via the Spotify Web API.

## [1.1.3] - 2026-02-23

### Fixed
- **Spotify Node Parameters**: Re-applied fixes for `Add New Tracks` and `Remove Existing Tracks` nodes to use correct parameter names (`playlistId`, `trackUris`) and operations (`add`, `Remove an item`) which were causing 400/403 errors and regressions.
- **Merge Paths**: Explicitly set `mode: append` and `numberOfInputs: 2` to ensure consistent behavior across n8n versions.

### Changed
- **Spotify API Strategy**: Investigated 403 Forbidden errors on `Add New Tracks` node. Documented an `HTTP Request` node alternative for direct API calls to help bypass potential n8n scope limitations.

## [1.1.2] - 2026-02-23

### Fixed
- **Collect Spotify URIs**: Rewrote the URI collection logic to correctly handle both flat and nested search result structures from the n8n Spotify node. Added validation to ensure only track URIs are collected (ignoring album/artist results) and normalized track IDs to URIs.
- **Spotify Search Node**: Adjusted search limit to 1 to simplify response processing and ensure a 1-to-1 mapping between searches and results, improving the reliability of the URI collection loop.

## [1.1.1] - 2026-02-23

### Fixed
- **Spotify Search Logic**: 
  - Improved search query accuracy by switching to `Artist Name Track Name` format and adding logic to strip "Remastered" and similar suffixes from track names.
  - Fixed a JSON syntax error in the `Search Track on Spotify` node that was causing it to be incorrectly defined.
  - Added `alwaysOutputData: true` to the search node to ensure subsequent nodes remain in sync even when no tracks are found.
  - Simplified the `limit` parameter structure to align with user feedback.

## [1.1.0] - 2026-02-23

### Fixed
- **Spotify Search**: Increased search limit to 10 in `additionalFields` to improve track matching reliability.
- **Spotify Playlist Operations**:
  - Corrected `Remove Existing Tracks` node to use the proper operation `Remove an item` and parameter names (`playlistId`, `trackUris`).
  - Updated `Add New Tracks` node to use correct parameter names (`playlistId`, `trackUris`).
- **Merge Logic**: Re-applied fixes to `Merge Paths` node to ensure it correctly handles mutually exclusive branches from the "Has Existing Tracks?" condition by disabling `waitForAllInputs`.

### Added
- Created `CHANGELOG.MD` to track project evolution.

## [1.0.1] - 2026-02-23

### Fixed
- **Merge Paths**: Changed mode to `append` and disabled `waitForAllInputs` to prevent the workflow from stalling when one branch of the `IF` node is skipped.
- **Spotify Nodes**: Initial correction of parameter structures for n8n Spotify node v1 (moving `limit` to `additionalFields`).

### Changed
- **Playlist Retrieval**: Implemented "Get Playlist" node as a workaround for 403 Forbidden errors encountered when using the "Get Tracks" operation due to n8n OAuth scope limitations.

## [1.0.0] - 2026-02-22

### Added
- Initial release of the Last.FM to Spotify Sync n8n workflow.
- Support for daily sync of top tracks (7-day, 1-month, etc.).
- Summary generation for sync operations.
