# Transport Dispatch Workbench — Downer Group Australia

Static HTML prototype mimicking the D365FO Transport Dispatch Workbench.

## Live site
Served from `DispatchWorkbench/preview/index.html`.

## Deploy to Azure Static Web Apps (via VS Code)

### Step 1 — Push this workspace to GitHub

Open a PowerShell terminal at the workspace root and run:

```powershell
cd 'c:\Users\spadakanti\Downloads\VSGITPERSO'
git init
git add .
git commit -m "Initial: Transport Dispatch Workbench prototype"
git branch -M main
# Create an empty repo on github.com first (no README, no .gitignore), then:
git remote add origin https://github.com/<your-user>/<your-repo>.git
git push -u origin main
```

### Step 2 — Create the Static Web App

1. Install the **Azure Static Web Apps** VS Code extension (publisher: Microsoft).
2. Sign in to Azure from the extension panel.
3. Click **+ Create Static Web App…**
4. When it asks for **Repository URL / Organization**, pick the GitHub org and the repo you just pushed. *(This is where you got the previous error — it needs a `https://github.com/...` URL, not a local path.)*
5. Branch: `main`
6. Presets: **Custom**
7. **App location:** `DispatchWorkbench/preview`
8. **Api location:** *(leave blank)*
9. **Output location:** *(leave blank — no build step)*

The extension commits a `.github/workflows/azure-static-web-apps-<name>.yml` file for you and triggers the first deployment. You'll get a URL like `https://<random-name>.azurestaticapps.net/`.

## Files

| Path | Purpose |
|------|---------|
| `DispatchWorkbench/preview/index.html` | The single-page prototype (entry point). |
| `DispatchWorkbench/preview/DispatchWorkbench.html` | Duplicate kept for direct-open convenience. |
| `DispatchWorkbench/preview/staticwebapp.config.json` | Azure SWA routing / cache config. |
| `DispatchWorkbench/DispatchWorkbench/` | D365FO metadata model (AxTable, AxForm, AxClass, etc.). |

## Local preview (no deploy)

```powershell
cd DispatchWorkbench\preview
python -m http.server 8080
# then open http://localhost:8080/
```

Or right-click `index.html` in VS Code → **Open with Live Server**.
