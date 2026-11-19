# JARVIS Tablet — Android APK + Python AI backend

This is a starter project for an installable Android tablet voice assistant.

## What is included
- Kivy Android app with an animated orb and chat transcript
- Android speech recognition + Android Text-to-Speech (via Pyjnius)
- "Hey Jarvis" wake phrase detection while the app is open and in the foreground
- Python FastAPI backend using the OpenAI API
- Invoice PDF generation endpoint
- Email-reply drafting endpoint (draft only; it does not send email)

## Important limitations
1. This starter listens for the wake phrase only while the app is open, foregrounded, and the screen/microphone are available. Android may stop microphone work in the background. Reliable always-on wake-word operation requires a native Android foreground service, persistent notification, battery-optimization handling, and Play Store policy review. Do not expect a browser-like app to listen after it is closed.
2. Voice processing is a turn-based pipeline: Android speech recognition -> Python backend -> AI text -> Android Text-to-Speech. It is not the low-latency OpenAI Realtime audio experience.
3. Keep the OpenAI API key on the backend. Never put it inside the APK.
4. The APK calls your backend over HTTPS. For local development, use a trusted HTTPS tunnel or deploy the backend. Android cleartext HTTP is disabled by default.
5. Email is drafted only. Review the text and send it yourself. Gmail sending can be added later with OAuth consent.

## 1. Backend setup (Windows/macOS/Linux)
Install Python 3.10 or newer.

```bash
cd backend
python -m venv .venv
# Windows:
.venv\\Scripts\\activate
# macOS/Linux:
source .venv/bin/activate
pip install -r requirements.txt
```

Set your API key as an environment variable:
```bash
# Windows PowerShell
$env:OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
# macOS/Linux
export OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
```

Run:
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

Deploy this backend to a host with HTTPS. Put the HTTPS base URL into `android_app/jarvis_config.py`.
Do not expose port 8000 publicly without HTTPS and appropriate access controls.

## 2. Build the Android APK
Buildozer works best on Linux. On Windows, use WSL2 Ubuntu. On macOS, use a Linux VM.

Install Buildozer and Android build dependencies following the official Buildozer installation guide:
https://buildozer.readthedocs.io/

Then:
```bash
cd android_app
buildozer android debug
```
The APK will be created under `android_app/bin/`.

For a release APK, configure signing and use Buildozer's release build instructions. Do not distribute a debug APK as a production app.

## 3. Configure the app
Edit `android_app/jarvis_config.py`:
```python
API_BASE_URL = "https://YOUR-HTTPS-BACKEND"
```

## 4. Permissions
The Buildozer spec requests RECORD_AUDIO and INTERNET. Android speech recognition and TTS are provided by the device's installed speech services. On first launch, grant microphone permission and ensure Google speech services are installed/enabled.

## 5. Wake phrase
Say "Hey Jarvis" when the app is showing the listening screen. The Android recognizer may time out or stop after a phrase; the app restarts recognition after each result. Tap the orb if the device speech service does not restart listening.

## 6. API endpoints
- `GET /health`
- `POST /chat` JSON: `{"message":"..."}`
- `POST /draft-email` JSON: `{"context":"...","tone":"Professional"}`
- `POST /invoice-from-command` JSON: `{"command":"Create an invoice for..."}` (AI extracts fields; missing details are not invented)
- `POST /invoice` JSON with `business_name`, `customer_name`, `items` (description, quantity, unit_price), optional `invoice_number`, `currency`

Invoices are generated as PDFs and returned as a file response.

## Safety and privacy
- The backend receives the text you speak and sends it to OpenAI for processing.
- Do not speak passwords, payment card details, or other secrets.
- The invoice endpoint creates a PDF from supplied values; verify tax, totals, and legal invoice requirements before use.


## Build from an Android tablet using GitHub Actions
1. Create a GitHub repository and upload the project files, including the `.github/workflows/build-apk.yml` workflow.
2. Open the repository's **Actions** tab. If prompted, enable Actions.
3. Select **Build JARVIS Android APK** and choose **Run workflow** (or push a change under `android_app/`).
4. Wait for the run to finish. Open the completed run and download the `jarvis-debug-apk` artifact.
5. Extract the downloaded artifact ZIP on the tablet and tap the APK to install. Android may ask you to allow installation from the browser/files app. Only install the APK built from your own repository.
