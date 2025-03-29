# Worldometer API

A REST API service that exposes real-time world data from Worldometer's live counters. This API provides up-to-date metrics such as current population figures, births, deaths, and net population growth, along with calculated change rates per minute.

## Features

- Real-time data from Worldometer's live counters
- Calculated change rates per minute for each metric
- Fallback data with clear indicators when live data is unavailable
- Rate limiting (1000 requests per hour per API key)
- Comprehensive error handling and logging

## API Documentation

The API documentation is available in the `documentation/api_documentation.yaml` file. The API provides a single endpoint:

- `GET /metrics`: Returns all available metrics with their current values and change rates

## Setup

1. Clone the repository
2. Create a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
4. Set up environment variables:
   - Copy `.env.example` to `.env`
   - Add your Supabase credentials and other required variables

## Development

The project uses FastAPI for the REST API implementation. To run the development server:

```bash
uvicorn app.main:app --reload
```

## License

[Add your license here] 