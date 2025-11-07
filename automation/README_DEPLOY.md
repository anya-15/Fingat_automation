FinGAT Deployment Notes (Render / Railway / Fly.io)

This document explains how to deploy the FinGAT automation bundle to a free cloud host (Render/Railway/Fly).

What we added
- Dockerfile (root) — builds an image based on n8n with Python installed and copies `automation/` and `requirements.txt`.
- render.yaml — Render service configuration that auto-deploys the repo and sets basic environment variables.

Important notes before deploying
1) Do NOT commit large checkpoints to GitHub. Keep your training artifacts out of the repo. Use S3 / Backblaze B2 / private storage and upload artifacts from your runner.

2) The Dockerfile uses the existing `requirements.txt` in the repo. If you want a smaller runtime, create a separate `requirements-deploy.txt` and update the Dockerfile's COPY and pip install lines.

3) Environment variables & secrets
   - Add the SMTP password and any API keys in the Render (or platform) dashboard as environment variables (do not hardcode in files).
   - Recommended: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY (for S3 uploads), N8N_SMTP_PASS

4) File storage and artifact discovery
   - The Dockerfile copies `automation/` into the image. For a running pipeline you will want the runner or the hosted environment to upload artifacts (top_stocks_YYYY-MM-DD.txt, automation_report.json, CKPT files) to a cloud storage (S3) and make them available to n8n via HTTPS or presigned URLs.

5) Testing locally before pushing
   - Build image: `docker build -t fingat-n8n .`
   - Run container (bind mount a local data folder):

     ```powershell
     docker run --rm -p 5678:5678 -v C:/path/to/your/repo/data:/app/data -v C:/path/to/your/repo/checkpoints:/app/checkpoints fingat-n8n
     ```

   - Visit http://localhost:5678 and login with admin / fingat2025 (or whatever creds you set via env)

Next steps (recommended)
- Implement a runner (Render background worker or GitHub Actions) to run fetch->train->predict and upload artifacts to S3.
- Update `automation/n8n_workflow.json` to use HTTP Request nodes and read `automation_report.json` (or presigned URLs) instead of local-file reads and PowerShell nodes.

If you want, I can now:
- Create a runner script that uploads artifacts to S3 and produces `automation_report.json` (Option: use boto3 or S3-compatible API),
- Or produce a cloud-ready `n8n_workflow.json` version which fetches artifacts by URL rather than reading local files.

Tell me which of those you'd like next.