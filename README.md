# HomePod Climate Log

Free temperature and humidity log for a HomePod mini (or HomePod 2), using only GitHub.

```
HomePod (Home automation) ──POST──▶ GitHub API ──▶ GitHub Actions ──▶ readings.csv (data branch)
                                                                           │
                                       GitHub Pages website  ◀──reads──────┘
```

- **Cost:** $0. The repo must be **public**, because GitHub Actions minutes and GitHub Pages are free for public repos. Anyone with the link can see your readings.
- **Delay:** the page shows a new reading within about 5 minutes.

---

## 1. Create the repo

1. On GitHub, create a new **public** repository, e.g. `homepod-climate-log`.
2. Upload these files, keeping the folder structure:
   - `index.html`
   - `.github/workflows/log-reading.yml`
   - `README.md`

## 2. Turn on the website

Repo **Settings → Pages → Build and deployment**:
- Source: **Deploy from a branch**
- Branch: **main**, folder **/ (root)** → Save

Your page will be at `https://YOUR-USERNAME.github.io/homepod-climate-log/`.

## 3. Make a token for the HomePod

GitHub **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**:
- Repository access: **Only select repositories** → pick this repo
- Permissions → Repository permissions → **Contents: Read and write**
- Expiration: the longest you're comfortable with (set a reminder to renew it)

Copy the token (starts with `github_pat_`). It can only touch this one repo.

## 4. Test it (optional, before the HomePod part)

Repo **Actions → Log climate reading → Run workflow** and enter a test temperature and humidity.
After about 30 seconds a `data` branch with `readings.csv` appears, and the reading shows on your page.

Or from a computer:

```bash
curl -X POST https://api.github.com/repos/YOUR-USERNAME/homepod-climate-log/dispatches \
  -H "Authorization: Bearer github_pat_XXXX" \
  -H "Accept: application/vnd.github+json" \
  -d '{"event_type":"climate","client_payload":{"temperature":75.2,"humidity":58,"room":"Living Room","unit":"F"}}'
```

A blank reply (HTTP 204) means it worked.

## 5. Set up the HomePod automation

Do this on an iPhone or iPad in the **Home** app (the automation then runs on your home hub, so your phone doesn't need to be home):

1. **Automation** tab → **+** → **A Time of Day Occurs** → pick a time, every day → Next.
2. Scroll to the bottom → **Convert to Shortcut**.
3. Add **Get the state of** → choose the HomePod's **Temperature** sensor.
4. Add **Get the state of** → choose the HomePod's **Humidity** sensor.
5. Add **Get Contents of URL**:
   - URL: `https://api.github.com/repos/YOUR-USERNAME/homepod-climate-log/dispatches`
   - Method: **POST**
   - Headers:
     - `Authorization` = `Bearer github_pat_XXXX`
     - `Accept` = `application/vnd.github+json`
   - Request Body: **JSON**
     - `event_type` (Text) = `climate`
     - `client_payload` (Dictionary):
       - `temperature` (Number) = *State* from step 3
       - `humidity` (Number) = *State* from step 4
       - `room` (Text) = `Living Room`
       - `unit` (Text) = `F` (or `C` if your Home app shows Celsius)
6. Done.

**How often it logs:** each Home "time of day" automation fires once a day, so duplicate the automation for each time you want a reading (for example every 2 or 3 hours). In the Home app you can also trigger on **A Sensor Detects Something → Temperature rises above / drops below** to log when it changes.

**More than one HomePod:** add another pair of "Get state" actions and another "Get Contents of URL" with a different `room` name. Each room gets its own color on the chart.

---

## Notes

- The HomePod sometimes reports `0` or nothing. The workflow drops impossible values instead of logging them.
- The sensor sits inside a warm speaker, so treat readings as trends rather than lab-grade numbers.
- Readings are stored on a separate `data` branch so the website doesn't redeploy for every reading.
- To change the page's repo detection (for example on a custom domain), set `CONFIG.repo` near the top of the script in `index.html`.
