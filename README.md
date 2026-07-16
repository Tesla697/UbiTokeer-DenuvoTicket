# UbiTokeer DenuvoTicket — Ubisoft Token Generator GUI

A clean Windows GUI tool to extract Denuvo authentication tickets and ownership tokens from Ubisoft/Uplay accounts.

## What's new in v1.2.0

- **Config Files (Cheese) tab**: Bundled per-game config packs (AC Shadows, AC IV: Black Flag, PoP: The Lost Crown, Star Wars Outlaws). One-click **Extract pack** drops `DannyTokenReq.exe` + `upc_r2.ini` + the Uplay loader wherever you choose. Antivirus-quarantine detection reports exactly which files were removed and how to whitelist them.
- **Drag & drop / paste request tokens**: Drop a `token_req_*.txt` onto the request box (or paste `token|appid` directly). The `|appid` suffix is stripped, the clean token loads, and the **App ID auto-fills** — you just press *Generate token*. Falls back to the filename for the appid when there's no suffix.
- **EXCEEDED ACTIVATIONS reporting**: When an account is out of tokens for a game, the tool shows a clear "EXCEEDED ACTIVATIONS" banner + dialog instead of a raw error code. Other Ubisoft result codes (NotOwned / NoSessions / TimeOut / ServerError) now get plain-English messages too.
- **Uplay Game ID lookup link**: Direct link to [Haoose/UPLAY_GAME_ID](https://github.com/Haoose/UPLAY_GAME_ID) so you can find exact game/DLC IDs.
- **Automatic updates**: On launch the app checks this repo's latest GitHub Release and offers a one-click self-update.
- **UI fixes**: Corrected window sizing, readable config text, and pinned login controls that no longer scroll off-screen.

## Features

- **Account Management**: Store and manage multiple Ubisoft accounts locally
- **Remember-Me Support**: Persist login sessions with device anchoring to avoid 409 Conflict errors
- **2FA Handling**: Built-in two-factor authentication flow
- **GUI Interface**: Clean Windows Forms UI for easy token generation
- **Batch Token Generation**: Generate Denuvo tickets for any appid you own
- **Token Export**: Save tokens directly to JSON or copy to clipboard
- **Auto-Update**: Checks GitHub Releases on launch and updates itself in place

## Requirements

- Windows 10 or later
- .NET 8.0 Runtime
- Valid Ubisoft account with game ownership

## Installation & Usage

1. Download `DenuvoTicketGUI.exe` from the latest release
2. Run it — no installation or dependencies needed (fully self-contained)
3. That's it

### Workflow

1. **Add or Select Account**
   - Click "Add account" to login with new credentials
   - Or select an existing account from the list

2. **Authenticate**
   - Click "Log in to selected account"
   - Complete 2FA if prompted (paste code from your authenticator)
   - Device will be anchored to prevent session conflicts

3. **Generate Token**
   - Enter the **appid** of a game you own (e.g., `4740` for Avatar: Frontiers of Pandora)
   - Click "Generate token"
   - Wait for the operation to complete (~30 seconds)
   - Token output appears in the text area

4. **Save or Copy**
   - Tokens auto-save to `Ubisoft_Token_<UUID>/dbdata.json` in the output directory
   - Use "Copy output" to copy token to clipboard

---

## Config Files (Denuvo Request Token)

The **Denuvo request token** field in the GUI is optional. Leave it empty and the tool
stops at the `OwnershipToken`. Fill it in and the tool also mints the `GameToken` and
`OwnershipListToken` that the DRM actually consumes.

That request token is **produced per-game by `DannyTokenReq.exe`**, which reads a
per-game config file called **`upc_r2.ini`**. You place the config next to the tool,
run it, and paste its output string into the GUI.

### Where the config files go

Each game ships as a "Cheese" folder that already contains the tool + its config.
The layout the tool expects:

```
<GameName>/
├── DannyTokenReq.exe        <- the request-token generator
├── upc_r2.ini               <- CONFIG FILE (edit this)
├── upc_r2_loader64.dll      <- Ubisoft Connect emulation loader
├── dbdata.dll               <- reads dbdata.json produced by DenuvoTicketGUI
├── steam_api64.dll
├── steamclient64.dll
└── avatar.png               <- referenced by Avatar= in the ini
```

> **Note:** For some titles (e.g. *Prince of Persia: The Lost Crown*) the config lives
> next to the game's real loader instead:
> `<Game>_Data/Plugins/x86_64/upc_r2.ini`. Put `DannyTokenReq.exe` in that same folder
> when generating.

### The `upc_r2.ini` config format

```ini
[Settings]
Username   = LuaTools                                   ; display name (cosmetic)
Email      = you@example.com                            ; account email (cosmetic)
UserId     = 782028f3-a3ca-4b2f-a56f-53c721cf6a72       ; Ubisoft account UUID
GameId     = 66333333-e688-4d1f-b693-39267e890df2       ; Uplay GameId for the title
Language   = en-US
Avatar     = avatar.png                                 ; png, 64/128/256 px square

; Save location: 0 = %AppData%\Goldberg UplayEmu Saves | 1 = SavePath in game folder | 2 = custom
SaveType      = 1
SavePath      = saves
SaveExtension = .save

[DLC]          ; one DLC product id per line — everything you want unlocked
64208
64209
64210
...

[Items]        ; is_ulc in-game unlockables (optional)
8008
8009

[Chunks]       ; installer pieces — fill from uplay_install.manifest or leave empty
```

**Key fields you must get right:**

| Field | What it is | How to fill it |
|-------|-----------|----------------|
| `UserId` | Ubisoft account UUID | The account you logged into in DenuvoTicketGUI |
| `GameId` | Uplay GameId of the title | Per-game (see table below) |
| `[DLC]` | DLC product ids to include | Every add-on id you want the token to claim ownership of |
| `[Items]` | `is_ulc` unlockables | Optional in-game items |
| `[Chunks]` | Installer chunk ids | Auto-fill with `--manifest uplay_install.manifest`, or leave empty |

### Included game configs

| Game | Config location | GameId | DLC ids |
|------|-----------------|--------|---------|
| **AC Shadows** | `Ac Shadows Cheese/upc_r2.ini` | `66333333-e688-4d1f-b693-39267e890df2` | 64208–64628 + Items 8008–8012, 64527/64528 |
| **AC IV: Black Flag** | `Black Flag Cheese/upc_r2.ini` | `c7ddd369-1202-4699-abea-abbe6ec895f6` (product `66088`) | 66233–67586 |
| **PoP: The Lost Crown** | `POP_LostCrown_DLC/TheLostCrown_Data/Plugins/x86_64/upc_r2.ini` | `66333333-e688-4d1f-b693-39267e890df2` | 6155–64699 |
| **Star Wars Outlaws** | `Star Wars Outlaws/.../upc_r2.ini` | `66333333-e688-4d1f-b693-39267e890df2` | 157–63745 |

### Workflow: config → request token → GUI

1. **Edit the config** — open the game's `upc_r2.ini`, set `UserId` to the account you'll
   use in DenuvoTicketGUI, confirm `GameId` and the `[DLC]` list.
2. **Generate the request token** — run `DannyTokenReq.exe` from that folder. It reads
   `upc_r2.ini`, drives `upc_r2_loader64.dll`, and emits the game's Denuvo **request
   token** (the per-machine activation request).
   - If `[Chunks]` needs filling: `DannyTokenReq.exe --manifest uplay_install.manifest`
3. **Copy the request token** it prints/writes.
4. **Paste it into DenuvoTicketGUI** — drop it into the *"Denuvo request token"* box,
   enter the matching **appid**, and click **Generate token**.
5. **Result** — the GUI mints `GameToken` + `OwnershipListToken` and saves
   `Ubisoft_Token_<UUID>/dbdata.json`.
6. **Deploy** — place that `dbdata.json` where `dbdata.dll` in the game folder reads it,
   and the title activates offline against the minted tokens.

## Troubleshooting

### 409 Conflict Error

If login fails with `HttpRequestException: Request failed with status code Conflict (409)`:

1. **First Time**: This is normal on first login. The tool registers a device ID.
2. **Session Collision**: Delete the account and re-login to refresh credentials
3. **Wait & Retry**: Ubisoft sometimes locks sessions temporarily; wait 5 minutes and try again

### 2FA Code Prompt

- Check your authenticator app (Microsoft Authenticator, Google Authenticator, Authy, etc.)
- Paste the 6-digit code into the prompt window
- Code expires after 30 seconds; get a fresh one if it times out

### Token Not Saving

- Check the output directory path at the bottom of the GUI
- Ensure you have write permissions to that folder
- Use "Browse..." to change the save location

## Output Format

Tokens are saved as:
```
Ubisoft_Token_<ID>.json
```

The JSON contains:
- `DenuvoTicket` — Denuvo protection ticket
- `GameToken` — Game-specific authentication token
- `OwnershipListToken` — List of your owned games

## What's Included

```
DenuvoTicketGUI.exe   (~150 MB)  — Fully self-contained executable
                                    All dependencies bundled, no runtime installation needed
```

That's it. Just one file.

## Output Format

Tokens are saved as:
```
Ubisoft_Token_<UUID>/
└── dbdata.json
```

The `dbdata.json` contains:
```json
{
  "DenuvoToken": "<authentication ticket for Denuvo>",
  "OwnershipListToken": "<list of games owned on this account>",
  "DLCIds": [<array of DLC IDs>]
}
```

Each user gets a unique `Ubisoft_Token_<UUID>` folder in the output directory.

## Notes

- Accounts and tokens are stored locally — never transmitted to third parties
- Remember-me tickets persist across sessions for quick re-authentication
- Device anchoring significantly reduces 409 errors by preventing session conflicts
- Two-factor authentication is required by Ubisoft and fully supported

## License

For educational and personal use only.
