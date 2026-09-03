# NLP Sentiment Analysis with BERT

A small Flask web application that sends text to the Watson sentiment-analysis service used by the IBM Developer Skills Network lab. The service returns a sentiment label and score, which the application displays in the browser.

## Features

- Browser form for entering text.
- Sentiment analysis through the external `SentimentPredict` service.
- Simple Flask endpoint that returns a human-readable result.
- Unit test examples for positive, negative, and neutral text.

## Architecture

```text
Browser
	|
	| GET /sentimentAnalyzer?textToAnalyze=...
	v
Flask application (server.py)
	|
	| POST JSON: {"raw_document": {"text": "..."}}
	v
Watson sentiment service
```

- `server.py` creates the Flask app, serves the page, and exposes the API routes.
- `templates/index.html` contains the Bootstrap-based user interface.
- `static/mywebscript.js` calls the Flask sentiment endpoint and writes the response to the page.
- `SentimentAnalysis/sentiment_analysis.py` calls the external service and normalizes its response to `label` and `score`.
- `test_sentiment_analysis.py` contains `unittest` checks using the live sentiment service.
- `SentimentAnalysis/__init__.py` marks the analysis directory as a Python package.

## Technologies

- Python 3
- Flask
- Requests
- Python `unittest`
- Bootstrap 4.3.1 loaded from the StackPath CDN in the page
- External Watson BERT sentiment-analysis service

## Requirements

- Python 3 is required.
- Internet access is required both to run sentiment analysis and to run the tests, because requests are sent to the external Watson service.
- No database, local model, API key, or environment variable is configured by this repository.

## Installation

Clone the repository and create an isolated environment:

```bash
git clone https://github.com/ibm-developer-skills-network/zzrjt-practice-project-emb-ai.git
cd zzrjt-practice-project-emb-ai
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install Flask requests
```

On Windows PowerShell, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

There is currently no `requirements.txt` or other dependency lockfile. The two packages above are the runtime dependencies imported by the application and tests.

## Configuration

There is no application configuration file or environment-variable configuration. The integration is defined in `SentimentAnalysis/sentiment_analysis.py`:

- Service URL: `https://sn-watson-sentiment-bert.labs.skills.network/v1/watson.runtime.nlp.v1/NlpService/SentimentPredict`
- Model header: `grpc-metadata-mm-model-id: sentiment_aggregated-bert-workflow_lang_multi_stock`

The service request body is:

```json
{
	"raw_document": {
		"text": "I love working with Python"
	}
}
```

The application does not persist requests or results. Do not treat the current endpoint as production-ready: it has no authentication, rate limiting, request timeout, structured error response, or input validation beyond the upstream service response.

## Run Locally

From the repository root, with the virtual environment activated:

```bash
python server.py
```

The development server binds to `0.0.0.0` on port `5000`. Open <http://localhost:5000/> in a browser. Stop it with `Ctrl+C`.

## API

### `GET /`

Returns the browser interface from `templates/index.html`.

### `GET /sentimentAnalyzer`

Query parameter:

| Parameter | Required | Description |
| --- | --- | --- |
| `textToAnalyze` | Yes | Text sent to the sentiment-analysis service. |

Example:

```bash
curl --get 'http://localhost:5000/sentimentAnalyzer' \
	--data-urlencode 'textToAnalyze=I love working with Python'
```

Successful response:

```text
The given text has been identified as POSITIVE with a score of 0.99.
```

The exact score is returned by the upstream service. If the upstream service returns an unsupported or error response, the application returns `Invalid input! Try again.` as plain text.

## Testing

The test module calls the live upstream service and expects the labels `SENT_POSITIVE` and `SENT_NEGATIVE` for its positive and negative examples. Run it directly from the repository root:

```bash
python test_sentiment_analysis.py
```

The neutral example currently calls the analyzer but does not assert a result. Tests can fail when the external service is unavailable, changes its model output, or network access is blocked. The repository does not include mocked-service tests.

## Build and Deployment

There is no build step: this is a server-rendered Flask application with static JavaScript and HTML assets. The repository contains no WSGI configuration, `Procfile`, Dockerfile, or deployment workflow. A hosting platform that runs Python must install Flask and Requests and start the application with `python server.py` (or an equivalent process command). The application listens on port `5000`; platforms that provide a dynamic port will require a code/configuration change because the current server does not read a `PORT` environment variable.

The Watson service URL is specific to the Skills Network lab and may not be suitable for production deployment. Confirm that it is reachable from the hosting platform before deploying.

## Deploy the Frontend to GitHub Pages

GitHub Pages hosts static files only. It cannot run `server.py`, so it cannot host the complete application by itself. The current frontend also calls `sentimentAnalyzer` as a relative URL, which means it expects the Flask API to be served by the same origin. To use GitHub Pages, host the Flask API separately and configure the frontend to call that public API URL; that frontend code change is not included in this documentation-only update.

For a static frontend deployment after making that API URL change:

1. Push the repository to GitHub. The repository name used by these instructions is `zzrjt-practice-project-emb-ai`.
2. In the repository on GitHub, open **Settings**, then **Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**.
4. Select the branch containing the frontend and choose the folder that contains the static site files. GitHub Pages cannot publish the repository's `templates/index.html` as a Flask template automatically; place the frontend entry point in the selected Pages folder as part of the frontend deployment preparation.
5. Click **Save** and wait for the Pages deployment workflow to complete.
6. Open the published Pages URL shown in **Settings > Pages** and verify that the browser can reach the separately deployed Flask API.

The GitHub Pages site alone will not provide working sentiment analysis unless the API is deployed elsewhere and the browser is allowed to call it by the API's CORS policy. Do not place private credentials in the static frontend.

## Project Structure

```text
.
├── server.py
├── test_sentiment_analysis.py
├── SentimentAnalysis/
│   ├── __init__.py
│   └── sentiment_analysis.py
├── static/
│   └── mywebscript.js
├── templates/
│   └── index.html
├── .gitignore
├── LICENSE
└── README.md
```
