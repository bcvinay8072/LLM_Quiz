DEPLOY TO HUGGING FACE SPACES
=============================

This file describes the exact steps to deploy this repository to a Hugging Face Space using the included `Dockerfile` and how to configure required secrets.

1) Important: rotate exposed keys
- If you previously committed `.env` with any secrets (Google API keys, SECRET, etc.), rotate those secrets now (do not reuse them).

2) Commit the changes you just made locally
```powershell
Set-Location 'C:\Users\bcvpt\OneDrive\Documents\LLM_QUIZ\LLM-Analysis-TDS-Project-2'
git add Dockerfile DEPLOY_TO_SPACES.md
git rm --cached .env -ErrorAction SilentlyContinue
git commit -m "Prepare for HF Spaces: update Docker base image; remove .env; add deploy guide"
git push origin main
```

3) Create a new Space (Web UI)
- Go to: https://huggingface.co/spaces/new
- Enter a Space name, set "SDK" to "Docker" (or select Docker when available), and create the Space.

4) Add Secrets in Space settings
- In the Space web UI, open `Settings` -> `Secrets` and add the following variables (exact names used by the app):
  - `EMAIL` — your email value
  - `SECRET` — secret string used by `/solve` endpoint
  - `GOOGLE_API_KEY` — Google API key (if required)
  - Any other keys you use (e.g., `AI_PIPE_KEY`, `AI_PIPE_URL`)

5) Connect the GitHub repo to the Space (or push directly)
- Option A: Link the GitHub repository to the Space (preferred). The Space will pull from your GitHub repo and build.
- Option B: Push directly to the Space remote:
```powershell
# Replace <hf-username> and <space-name>
git remote add hf "https://huggingface.co/spaces/<hf-username>/<space-name>"
git push hf main
```

6) Build and runtime notes
- The `Dockerfile` installs Playwright and Chromium and also Tesseract; builds can be large and may take several minutes.
- Ensure the Space has sufficient build timeout/size; if you hit build limits, consider using a smaller Playwright image or prebuilt browser layers.
- The app listens on port `7860` (exposed in the Dockerfile). Spaces expects your container to listen on that port.

7) Test the deployed endpoint
```powershell
$body = @{
  email  = 'your-email@example.com'
  secret = 'your-secret'
  url    = 'https://tds-llm-analysis.s-anand.net/demo'
} | ConvertTo-Json

Invoke-RestMethod -Uri 'https://<hf-username>-<space-name>.hf.space/solve' -Method Post -ContentType 'application/json' -Body $body
```

8) Troubleshooting
- If build fails due to Playwright, inspect build logs in the Space CI. Try replacing the Playwright install step with the official Playwright Docker base image if necessary.
- If your app needs additional apt packages during build, add them to the `apt-get install` list in `Dockerfile`.

If you want, I can also create a small GitHub Actions workflow to automatically push to the Space on every `main` push (requires HF token). Ask me to add it.
