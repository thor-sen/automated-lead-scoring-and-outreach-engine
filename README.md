# GTM Automation Engine

A production-grade GTM automation engine built on live HubSpot data. Data flows from CRM pull through enrichment, ML scoring, composite prioritization, territory routing, and AI-driven outreach generation. Each stage reads from and writes back to HubSpot, so the full pipeline produces real CRM changes — scored companies, assigned owners, prioritized tiers, and generated outreach messages visible to reps in their workflows. The system is built for a healthcare vertical (health system partnerships) but the architecture generalizes to any B2B sales motion with firmographic scoring, pain signal detection, and territory-based routing.

## Architecture

```
HubSpot CRM
    │
    ▼
hubspot-data-connector ──► lead-enrichment-pipeline ──► ml-lead-scoring-model
                                                              │
                                                              ▼
ai-outreach-engine ◄── territory-routing-service ◄── composite-lead-scorer
    │
    ▼
gtm-intelligence-dashboard
```

**Data flows left to right, top to bottom:**

1. **hubspot-data-connector** pulls raw company, contact, and deal records from HubSpot
2. **lead-enrichment-pipeline** enriches companies with firmographic data from People Data Labs
3. **ml-lead-scoring-model** trains a logistic regression model on firmographic features and scores real companies as ICP fit (0-100)
4. **composite-lead-scorer** combines firmographic, engagement, and pain signal scores into a weighted composite and assigns priority tiers
5. **territory-routing-service** assigns companies to reps based on territory rules and company size via webhook
6. **ai-outreach-engine** uses Claude to classify ICP fit, detect intent from pain signals, and generate personalized BDR outreach
7. **gtm-intelligence-dashboard** surfaces everything in a Streamlit dashboard with filtering, routing audit, and pipeline projections

## Components

| Component | Description |
|-----------|-------------|
| [hubspot-data-connector](https://github.com/thor-sen/hubspot-data-connector) | Pulls all company, contact, and deal records from HubSpot with automatic pagination and rate limit handling |
| [lead-enrichment-pipeline](https://github.com/thor-sen/lead-enrichment-pipeline) | Enriches companies via PDL API with employee count, revenue, industry, and tech stack |
| [ml-lead-scoring-model](https://github.com/thor-sen/ml-lead-scoring-model) | Trains a logistic regression model on firmographic features and scores real HubSpot companies (0-100) |
| [composite-lead-scorer](https://github.com/thor-sen/composite-lead-scorer) | Computes weighted composite scores (firmographic 50%, engagement 25%, pain signals 25%) and assigns priority tiers |
| [territory-routing-service](https://github.com/thor-sen/territory-routing-service) | FastAPI webhook service that routes new companies to reps by territory and company size |
| [ai-outreach-engine](https://github.com/thor-sen/ai-outreach-engine) | AI BDR pipeline using Claude for ICP classification, intent detection, and outreach generation |
| [gtm-intelligence-dashboard](https://github.com/thor-sen/pipeline-intelligence-dashboard) | Streamlit dashboard for company filtering, routing audit, and pipeline health projections |

## What's Real vs. Stubbed

The HubSpot integration is fully live — every component reads from and writes back to a real HubSpot portal with real company data. The CRM data connector, territory router, composite scorer, ML scoring model, and dashboard all run against production CRM records.

Two areas are partially stubbed. The lead enrichment pipeline is fully implemented but requires a valid People Data Labs API key to return enrichment results — without one, companies are marked as failed and the HubSpot pull and write-back still run independently. The pain signal detector in the AI outreach engine uses mocked news data (realistic healthcare articles for a small set of known companies) rather than a live news API. The AI classification, gating logic, and outreach generation are real Claude calls against real CRM fields.

## Tech Stack

- **Languages:** Python, SQL
- **AI/ML:** Anthropic Claude (claude-sonnet-4-5), scikit-learn (LogisticRegression)
- **Web framework:** FastAPI + Uvicorn
- **Dashboard:** Streamlit
- **APIs:** HubSpot CRM v3, People Data Labs v5
- **Database:** PostgreSQL (pipeline analytics)
- **Libraries:** requests, pandas, pydantic, python-dotenv, BeautifulSoup4, psycopg2

## Setup

Each component has its own README with setup instructions, `.env` requirements, and how to run. Start with the [hubspot-data-connector](hubspot-data-connector/) to pull data, then work through the pipeline in order.

All components require a `.env` file with at minimum:

```
HUBSPOT_API_KEY=pat-na1-your-token-here
```

Components using Claude also require:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```
# gtm-automation-engine
