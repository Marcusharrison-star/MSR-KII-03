# How to Run EcoGen Diagnostics

This guide provides instructions for setting up, running, testing, and deploying **EcoGen Diagnostics** locally and on Vercel.

---

## 1. Prerequisites

- **Python**: Version 3.10, 3.11, or 3.12 installed on your machine (`python3 --version` or `python --version`)
- **pip**: Standard Python package installer
- **Git** (optional): For repository management and Vercel GitHub integration
- **Vercel CLI** (optional): For deploying directly from your terminal

---

## 2. Unpacking the Project

Extract the downloaded `ecogen_diagnostics.zip` archive:

```bash
unzip ecogen_diagnostics.zip
cd ecogen-diagnostics
```

The directory structure:
```text
ecogen-diagnostics/
├── api/
│   ├── __init__.py
│   ├── analysis.py        # Core genetics calculation engine & CSV validation
│   └── index.py           # Vercel Flask entrypoint exporting `app`
├── public/
│   ├── index.html         # Responsive, accessible single-page interface
│   ├── app.js             # Client controller with safe parsing & error states
│   └── style.css          # Theme styling, focus states & reduced-motion support
├── tests/
│   ├── test_analysis.py   # Unit tests for calculations, missing calls & edge cases
│   └── test_api.py        # Integration tests for /api/health and /api/upload
├── requirements.txt       # Pinned dependencies (Flask==3.0.3, pandas==2.2.2, Werkzeug==3.0.3)
├── vercel.json            # Vercel route rewrites (destination: /api)
├── run_tests.py           # Unified test runner
├── README.md              # Project documentation
└── HOW_TO_RUN.md          # Setup & execution guide
```

---

## 3. Local Installation & Setup

It is recommended to use a Python virtual environment:

### Step 1: Create and Activate a Virtual Environment
- **On macOS / Linux**:
  ```bash
  python3 -m venv venv
  source venv/bin/activate
  ```
- **On Windows (Command Prompt / PowerShell)**:
  ```cmd
  python -m venv venv
  venv\Scripts\activate
  ```

### Step 2: Install Required Dependencies
Install the exact pinned versions:
```bash
pip install -r requirements.txt
```

This installs:
- `Flask==3.0.3`
- `pandas==2.2.2`
- `Werkzeug==3.0.3`

---

## 4. Running the Automated Tests

Run the test suite to verify the calculation engine and API endpoints:

```bash
python run_tests.py
```

All 17 tests verify:
- Allele pooling and per-locus $H_o$ and $H_e$ calculations
- Missing-call pairwise omission and empty-locus rejection
- Missing partner column rejection (HTTP 400)
- Global mean $H_o$, $H_e$, and $F_{is}$ calculation
- Monomorphic zero-heterozygosity edge case handling ($F_{is} \to \text{null}$, rendered as `N/A`)
- Classification logic (`CRITICAL` [red], `WARNING` [yellow], `STABLE` [green])
- Health check endpoint (`GET /api/health`) and upload endpoint (`POST /api/upload`)

---

## 5. Running the Application Locally

You can run the application locally using Flask's development server or a local static file server.

### Option A: Using the Local Flask Runner (Recommended)

Run the backend and static file server directly:
```bash
python -c "from api.index import app; app.run(host='127.0.0.1', port=5000, debug=True)"
```

To serve both the API and the static files in `public/` locally with Flask, you can add a simple local runner `local_dev.py`:
```python
from api.index import app
from flask import send_from_directory

@app.route('/')
def serve_index():
    return send_from_directory('public', 'index.html')

@app.route('/<path:path>')
def serve_static(path):
    return send_from_directory('public', path)

if __name__ == '__main__':
    print("EcoGen Diagnostics running locally at: http://127.0.0.1:5000")
    app.run(host='127.0.0.1', port=5000, debug=True)
```
Run it with:
```bash
python local_dev.py
```
Open your browser to:
```
http://127.0.0.1:5000
```

---

## 6. Deploying to Vercel

The repository is pre-configured for Vercel deployment with `vercel.json` routing `/api/(.*)` to `/api` (avoiding the `.py` static path issue):

```json
{
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "/api"
    }
  ]
}
```

### Deploying via Vercel CLI
```bash
npm install -g vercel
vercel
```

### Deploying via GitHub / Web Dashboard
1. Push the unzipped project folder to a GitHub repository.
2. In the Vercel Dashboard, import the repository with default settings:
   - **Framework Preset**: Other
   - **Root Directory**: `./`
   - **Output Directory**: `public/` (or default)
3. Click **Deploy**. Vercel detects `requirements.txt` and `api/index.py` and deploys the function at `/api`.