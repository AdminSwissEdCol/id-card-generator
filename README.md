# ID Card Generator

**Live tool:** https://adminswissedcol.github.io/id-card-generator/

Creates the student list and correctly named/cropped photos for ID card printing.
Photos are pulled from Chamilo LMS and synced nightly to Google Drive.

---

## How It Works

```
Chamilo LMS (live)
    ↓  sync_student_pictures.py  (nightly 02:00 or manual)
Google Drive → Metabase_Foto/
    ↓  index.html (Apps Script web app)
ID_Files_DATETIME/
    ├── ID_Cards_Output.xlsx
    └── Photos/  (413×531px JPEG, 35×45mm @ 300 DPI)
```

---

## Key Files & Locations

### This repo
| File | Purpose |
|------|---------|
| `index.html` | Frontend — runs in browser, calls Apps Script web app |
| `README.md` | This file |

### Apps Script (Google)
| Item | Location |
|------|---------|
| Apps Script project | [IDMakerTool spreadsheet](https://docs.google.com/spreadsheets/d/1D1JFXA5g9uam97Jauk4-QBwMGxB2rvVSh65bkclUvxE/edit?gid=1318531787#gid=1318531787) → Extensions → Apps Script |
| Web App URL | Stored by user in browser localStorage when first configured |
| Script file | `StudentIDCards.gs` — deploy as Web App: Execute as Me, Access Anyone |

### Google Drive (administration@swissedcol.ch)

**My Drive → Metabase_Foto**
https://drive.google.com/drive/folders/145WU-0WysmUJlEJ4mbXICSnQEJkEfsXA

**My Drive → Metabase_Foto → Student Pictures Directory**
(Col A = Name, Col B = Email, Col C = Drive photo link)
https://docs.google.com/spreadsheets/d/1SNPZ38GWm_N6dH_vrHV9U3yvfJjpaWAqM9F13jcNkc0/edit?gid=0#gid=0

**IDMakerTool** (Apps Script code + Web App URL)
https://docs.google.com/spreadsheets/d/1D1JFXA5g9uam97Jauk4-QBwMGxB2rvVSh65bkclUvxE/edit?gid=1318531787#gid=1318531787

### TrueNAS server scripts
| File | Path on server |
|------|---------------|
| Photo sync script | `/mnt/tank2/apps/transcript-api/sync_student_pictures.py` |
| API app | `/mnt/tank2/apps/transcript-api/app.py` |
| Requirements | `/mnt/tank2/apps/transcript-api/requirements.txt` |
| Nightly sync orchestrator | `/mnt/tank2/apps/chamilo-mirror/scripts/nightly_full_sync.sh` |
| Sync log | `/mnt/tank2/apps/chamilo-mirror/logs/nightly_sync.log` |

---

## Credentials & Tokens (Troubleshooting)

> ⚠️ Never commit actual secrets to git. This section documents **where** they live.

### Apps Script / Google Drive
| Secret | Location |
|--------|---------|
| Google service account JSON | `/mnt/tank2/apps/google-sync/lms-att-googl.json` |
| PyDrive settings (OAuth) | `/mnt/tank2/apps/google-sync/pydrive_settings.yaml` |
| Google Drive folder ID (`GDRIVE_FOLDER_ID`) | `docker-compose.yml` → `transcript-api` → environment |
| Pictures Sheet ID (`GSHEET_PICTURES_ID`) | `docker-compose.yml` → `transcript-api` → environment |

### Chamilo Live Database
| Secret | Location |
|--------|---------|
| Host | `lms.arch.swissedcol.ch` (also `170.10.161.154`) |
| Database | `lmsarchswissedco_s3` |
| User | `lmsarchswissedco_mirroruser` |
| Password | `/mnt/tank2/apps/secrets/src_db_password.txt` |
| Env vars in compose | `CHAMILO_DB_HOST`, `CHAMILO_DB_USER`, `CHAMILO_DB_PASS`, `CHAMILO_DB_NAME` |

### Mirror Database (local MariaDB)
| Secret | Location |
|--------|---------|
| Root password | `/mnt/tank2/apps/secrets/db_root_password.txt` |
| App user/pass | `mirroruser` / `mirrorpass` (in `docker-compose.yml`) |

### API Auth
| Secret | Location |
|--------|---------|
| `API_DATA_TOKEN` | `docker-compose.yml` → `transcript-api` → environment |
| `SHEETS_SYNC_TOKEN` | `docker-compose.yml` → `transcript-api` → environment |
| Cloudflare tunnel credentials | `/mnt/tank2/apps/secrets/cloudflared/` |

---

## Manual Photo Sync

Replace a student photo on Chamilo, then run immediately (no need to wait for nightly sync):

```bash
# On TrueNAS (as root)
docker exec transcript-api python /app/sync_student_pictures.py
```

Watch progress:
```bash
# The script logs directly to stdout — run interactively to see live output
```

---

## Nightly Sync Schedule

| Time | Job |
|------|-----|
| 02:00 | Full sync: Chamilo pull → transcripts → absences → attendance → photos |
| 03:20 | Export CSV reports |
| 03:25 | Rotate old exports |

---

## Related Repos

- **[lms-mirror-system](https://github.com/AdminSwissEdCol/lms-mirror-system)** — TrueNAS server scripts, Docker setup, `sync_student_pictures.py`, `app.py`
  - [sync_student_pictures.py](https://github.com/AdminSwissEdCol/lms-mirror-system/blob/main/transcript-api/sync_student_pictures.py)
