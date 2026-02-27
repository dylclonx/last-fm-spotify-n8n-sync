# Last.FM Top Tracks to Spotify Playlist Sync

This n8n workflow automatically syncs your most-listened tracks from Last.FM to a Spotify playlist every 24 hours.

## Workflow Overview

```
┌─────────────────┐    ┌───────────────────┐    ┌──────────────────────┐
│  Daily Trigger  │───▶│ ⚙️ Configuration  │───▶│ Get Last.FM Top      │
│  (every 24hrs)  │    │ (Edit settings)   │    │ Tracks               │
└─────────────────┘    └───────────────────┘    └──────────┬───────────┘
                                                           │
      ┌────────────────────────────────────────────────────┘
      ▼
┌───────────────────┐    ┌──────────────────────┐    ┌───────────────────┐
│ Process Track     │───▶│ Search Track on      │───▶│ Collect Spotify   │
│ Data (Clean names)│    │ Spotify              │    │ URIs              │
└───────────────────┘    └──────────────────────┘    └─────────┬─────────┘
                                                               │
      ┌────────────────────────────────────────────────────────┘
      ▼
┌───────────────────┐    ┌──────────────────────┐    ┌───────────────────┐
│ Wait Node         │───▶│ Get Playlist         │───▶│ Prepare Playlist  │
│ (Rate limiting)   │    │ (includes tracks)    │    │ Update            │
└───────────────────┘    └──────────────────────┘    └─────────┬─────────┘
                                                               │
      ┌────────────────────────────────────────────────────────┘
      ▼
┌───────────────────┐         ┌───────────────────┐
│ Has Existing      │───YES──▶│ Remove Existing   │───┐
│ Tracks?           │         │ Tracks (Bulk API) │   │
└─────────┬─────────┘         └───────────────────┘   │
          │ NO                                        │
          └──────────────────┬────────────────────────┘
                             ▼
┌───────────────────┐    ┌──────────────────────┐    ┌───────────────────┐
│ Merge Paths       │───▶│ Encode URIs for      │───▶│ Add New Tracks    │
│                   │    │ Spotify              │    │ (Bulk API)        │
└───────────────────┘    └──────────────────────┘    └─────────┬─────────┘
                                                               │
                                         ┌─────────────────────┘
                                         ▼
                               ┌───────────────────┐
                               │ Generate Summary  │
                               │                   │
                               └───────────────────┘
```

---

## Credentials Setup

### 1. Last.FM API Credential (HTTP Query Auth)

The Last.FM API key is stored securely using n8n's credential system.

#### Step 1: Get Your Last.FM API Key
1. Go to [Last.FM API Account Creation](https://www.last.fm/api/account/create)
2. Fill out the application form
3. Copy your **API Key**

#### Step 2: Create the Credential in n8n
1. In n8n, go to **Credentials** → **Add Credential**
2. Search for and select **"HTTP Query Auth"**
3. Configure it as follows:

| Field | Value |
|-------|-------|
| **Name** | `api_key` |
| **Value** | `your_lastfm_api_key_here` |

4. Give the credential a recognizable name like **"Last.FM API Key"**
5. Click **Save**

> ✅ This stores your API key securely — it won't be visible in the workflow JSON.

---

### 2. Spotify OAuth2 Credential

#### Step 1: Create a Spotify Developer App
1. Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Click **Create App**
3. Fill in the app details (name, description)
4. **Set the Redirect URI (Callback URL)**:

#### 🔗 Spotify Callback URL

| n8n Setup | Callback URL |
|-----------|--------------|
| **n8n Cloud** | `https://oauth.n8n.cloud/callback` |
| **Self-hosted** | `https://YOUR_N8N_DOMAIN/rest/oauth2-credential/callback` |
| **Local dev** | `http://localhost:5678/rest/oauth2-credential/callback` |

5. Add the callback URL to **Redirect URIs** in your Spotify app settings
6. Save and note your **Client ID** and **Client Secret**

#### Step 2: Create the Credential in n8n
1. In n8n, go to **Credentials** → **Add Credential** → **Spotify OAuth2**
2. Enter your **Client ID** and **Client Secret**
3. Click **Connect** to authorize with Spotify

---

## Workflow Setup

### Step 1: Import the Workflow
1. In n8n, go to **Workflows** → **Import from File**
2. Select the `LastFM to Spotify Sync n8n Workflow.json` file

### Step 2: Connect Credentials

| Node | Credential Type |
|------|-----------------|
| **Get Last.FM Top Tracks** | HTTP Query Auth → "Last.FM API Key" |
| **Search Track on Spotify** | Spotify OAuth2 |
| **Get Playlist** | Spotify OAuth2 |
| **Remove Existing Tracks** | Spotify OAuth2 |
| **Add New Tracks** | Spotify OAuth2 |

### Step 3: Configure Settings
Click on the **"⚙️ Configuration"** node and update:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `lastfm_username` | Your Last.FM username | `johndoe` |
| `spotify_playlist_id` | Playlist ID from Spotify URL | `37i9dQZF1DXcBWIGoYBM5M` |
| `track_limit` | Number of tracks to sync (1-100) | `25` |
| `time_period` | Time range for top tracks | `1month` |
| `api_wait_seconds` | Delay between API calls (seconds) | `31` |

#### Time Period Options:
| Value | Description |
|-------|-------------|
| `7day` | Past 7 days (weekly) |
| `1month` | Past month |
| `3month` | Past 3 months |
| `6month` | Past 6 months |
| `12month` | Past year |
| `overall` | All time |

### Step 4: Adjust Schedule (Optional)
Click on the **"Daily Trigger"** node to change how often the playlist updates.

### Step 5: Activate the Workflow
Toggle the workflow to **Active** and you're done!

---

## How It Works

1. **Schedule Trigger** → Executes every 24 hours (configurable)
2. **⚙️ Configuration** → Loads your settings and preferences
3. **Get Last.FM Top Tracks** → Fetches your top tracks using the API key
4. **Process Track Data** → Formats tracks for Spotify search and cleans track names (strips "Remastered", etc.)
5. **Search Track on Spotify** → Finds each track on Spotify using optimal search queries
6. **Collect Spotify URIs** → Gathers all found track URIs and tracks misses
7. **Wait before Get Playlist** → Optional delay to prevent rate limiting
8. **Get Playlist** → Retrieves playlist info to find existing tracks
9. **Prepare Playlist Update** → Extracts existing URIs for removal
10. **Has Existing Tracks?** → Checks if the playlist needs to be cleared
11. **Remove Existing Tracks** → Uses bulk API call to clear the playlist in one go
12. **Merge Paths** → Consolidates workflow after removal or skip
13. **Encode URIs for Spotify** → Prepares track URIs for the bulk add request
14. **Wait before Add** → Optional delay for API stability
15. **Add New Tracks** → Uses bulk API call to add all top tracks simultaneously
16. **Generate Summary** → Reports exactly what was synced and what was missing

> **Note:** Uses "Get Playlist" instead of "Get Tracks" to work around an n8n Spotify node scope limitation.

---

## Troubleshooting

### "Invalid redirect URI" error in Spotify
- Ensure the callback URL in your Spotify app **exactly matches** what n8n expects
- For n8n Cloud, use: `https://oauth.n8n.cloud/callback`
- Check for trailing slashes — they matter!

### "Invalid API key" error from Last.FM
- Verify you created an **HTTP Query Auth** credential (not Header Auth)
- Check that the **Name** field is exactly `api_key` (lowercase)
- Ensure the credential is selected in the "Get Last.FM Top Tracks" node

### "No tracks found" error
- Verify your Last.FM username is correct in the Configuration node
- Ensure you have recent scrobbles in the selected time period
- Test the API directly: `https://ws.audioscrobbler.com/2.0/?method=user.getTopTracks&user=YOUR_USERNAME&api_key=YOUR_KEY&period=7day&format=json`

### Spotify authentication errors
- Re-authenticate your Spotify credentials in n8n
- Verify the callback URL is correctly set in Spotify Developer Dashboard

### Some tracks not found on Spotify
- This is normal — not all tracks on Last.FM exist on Spotify
- The workflow will skip tracks that can't be found and report them in the summary

---

## Security Notes

✅ **API Key stored in credentials** — The Last.FM API key is stored in n8n's encrypted credential storage, not in the workflow JSON. This means:
- The key won't be exposed if you export/share the workflow
- The key is encrypted at rest in n8n
- You can update the key without modifying the workflow

---

## API References

- [Last.FM API - user.getTopTracks](https://www.last.fm/api/show/user.getTopTracks)
- [n8n Spotify Node Documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.spotify/)
- [n8n HTTP Query Auth](https://docs.n8n.io/integrations/builtin/credentials/httprequest/)
- [Spotify Web API - Playlist Endpoints](https://developer.spotify.com/documentation/web-api/reference/playlists)

## Notes

- The workflow **replaces ALL tracks** in the playlist each time (doesn't append)
- Uses **bulk API requests** for add/remove operations (faster and more reliable)
- Includes **Wait nodes** to handle Spotify API rate limiting gracefully
- Uses "Get Playlist" instead of "Get Tracks" to work around n8n scope limitations
- Maximum 100 tracks per sync (configurable via `track_limit`)
- Tracks are synced in order of play count (most listened first)
- Automatically **cleans track names** to improve Spotify search results
- Includes a **Quick Start Guide** sticky note directly inside the workflow
