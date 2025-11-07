Push instructions — create GitHub repo and deploy to Render (PowerShell)

This file contains copy-paste commands and step-by-step guidance for creating a GitHub repo, committing the current project (without large artifacts), and deploying to Render.

IMPORTANT: I cannot access your GitHub account or Render dashboard. Run these commands locally in PowerShell on your machine.

1) Clean up / verify large files
- Make sure large checkpoints and data are NOT staged for commit.
- If you previously committed checkpoint files, remove them from the index (do NOT delete them locally unless you want):

# remove from git index but keep files locally
git rm --cached -r checkpoints
git rm --cached -r data

# commit the removal
git commit -m "Remove large artifacts from index and add to .gitignore"

2) Initialize repository (if not already a git repo)

# from repo root (C:\Users\hp\Downloads\modern_fingat_2025)
cd C:\Users\hp\Downloads\modern_fingat_2025
git init

# add all files (gitignore prevents adding data/checkpoints)
git add .
git commit -m "Initial commit: FinGAT automation + deployment files"

3) Create the GitHub repo
Option A — using GitHub CLI (recommended if installed)
# install gh and authenticate first: https://cli.github.com/
# create a public repo named fingat-automation and push
gh auth login
gh repo create fingat-automation --public --source=. --remote=origin --push

Option B — create repo on github.com
- Go to https://github.com/new, name repo `fingat-automation`, create it (do NOT initialize README again if you already committed one locally).
- Then run these commands replacing <YOUR_REMOTE_URL> with the URL GitHub shows (HTTPS or SSH):

# example (HTTPS)
git remote add origin https://github.com/<your-username>/fingat-automation.git
git branch -M main
git push -u origin main

4) Verify files on GitHub
- Visit https://github.com/<your-username>/fingat-automation to confirm files are uploaded and that `data/` and `checkpoints/` are not present (they should be ignored by `.gitignore`).

5) Deploy to Render
- Sign in to https://render.com (create a free account if needed).
- New + → Web Service.
- Choose 'Connect a repository' and authorize GitHub if necessary, then choose your `fingat-automation` repo.
- Render will auto-detect `render.yaml` and propose a service. If not using render.yaml, configure:
  - Environment: Docker
  - Build Command: leave blank (Docker will build)
  - Start Command: leave blank (image runs `n8n start` as CMD)
- Under Environment → Add secret `N8N_SMTP_PASS` and any other secrets (AWS keys if you use S3).
- Click 'Create Web Service' and wait for the build & deploy logs.

6) After deployment
- Open the public URL Render gives (e.g., https://fingat-n8n.onrender.com)
- Login using the basic auth credentials set in `render.yaml` (admin / fingat2025) unless you changed them.

7) Final Render configuration
- In the Render dashboard → Environment, verify these keys are set:
  - N8N_SMTP_USER = Fingat393@gmail.com
  - N8N_SMTP_PASS = (your Gmail app password or SMTP pass)
  - N8N_SMTP_HOST = smtp.gmail.com
  - N8N_SMTP_PORT = 465
  - EXECUTIONS_MODE = regular

8) Test your n8n workflow
- In n8n, import `automation/n8n_workflow.json` (Workflows → Import from file) and enable it.
- Run the workflow manually (the Cron trigger node can also be manually triggered) to validate fetch → train → predict → email steps.

Troubleshooting tips
- Render build fails due to large Python wheel installs: consider creating a smaller `requirements-deploy.txt` listing only the runtime packages (yfinance, pandas, numpy, etc.) and remove heavy dev/testing tools (black, pytest) from the deploy requirements.
- If builds time out, try using a smaller base image or prebuilding wheels locally.
- If your runner needs GPU for full training: Render and free hosts don't provide GPUs. For full training, run on your laptop or a GPU provider.

Security notes
- Do NOT commit real credentials in any file. Always use Render environment variables or GitHub secrets.
- Do NOT upload production model checkpoints to a public GitHub repo.

If you want, I can now create the runner code that uploads artifacts to S3 and a cloud-ready `automation/n8n_workflow.cloud.json` that fetches artifacts by URL — reply "runner" or "workflow" to pick next. 
