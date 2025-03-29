# Backend Structure Document

## Introduction

This document explains how the backend for our real-time Worldometer data API is set up. The purpose of this backend is to expose live data from the Worldometer website through a single REST API endpoint. The API not only provides the latest figures such as current population, births, deaths, and net population changes but also calculates how fast these values are changing by comparing them to previously stored records. Every part of the system—from the way the live data is fetched using a headless browser to the way historical data is stored and compared—has been designed to work only when a request is made. This reduces unnecessary load on the worldometer website and keeps the system responsive and efficient.

## Data Source
This REST API uses the worldometer python library to get all of it's data. 
https://github.com/matheusfelipeog/worldometer
Documentation can be found in the python_api_documentation directory. 

We are providing access to the live counters in the worldometer python library via REST calls. The REST API is a way for web apps to access this python API. It's a proxy for the python API, a way for web apps to get access to the python APIs data. 

## API Design and Endpoints
The API REST interface has already been designed. Please consult the api_documentation.yaml file.

The API is designed as a single REST endpoint that returns all live counter metrics along with the computed change per minute. This endpoint is built using FastAPI and ensures that whenever a call is made, the system will trigger the headless browser session to fetch the latest data from the worldometer.info page. Once the current data is obtained, the API retrieves the most recent stored value from the Supabase database. It uses the difference between that value and the current value, along with the difference between the timestamp for that value and the timestamp for the current value, to calculate the change per minute. The response is structured in a clear JSON format, containing both the current metrics and an indicator flag that signifies whether the data returned is live or fallback data. This unified endpoint simplifies integration for developers who need real-time display data.

I *believe* that the python library uses a headless browser, so that after the first call it does not make requests to the worldometer website to get data.
However you should confirm this. 

## Backend Architecture
At the heart of the system is a Python-based backend powered by the FastAPI framework. 

## Database Management and Schemas
The system uses Supabase as its database management solution. 

### Database Tables

#### 1. Metrics Table
Stores historical values for each metric with their change rates.

```sql
CREATE TABLE metrics (
    key VARCHAR(50) NOT NULL,
    value NUMERIC NOT NULL,
    when TIMESTAMP WITH TIME ZONE NOT NULL,
    change_per_minute NUMERIC NOT NULL,
    PRIMARY KEY (key, when)
);
```

**Columns:**
- `key`: The metric identifier (e.g., "WORLD_POPULATION")
- `value`: The metric's value at the timestamp
- `when`: UTC timestamp when the value was recorded
- `change_per_minute`: The calculated rate of change per minute

**Indexes:**
- Primary key on (key, when)
- Index on (key) for efficient lookups of latest values

#### 2. API_Keys Table
Stores valid API keys and associated user information.

```sql
CREATE TABLE api_keys (
    api_key UUID NOT NULL,
    email_address VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    PRIMARY KEY (api_key)
);
```

**Columns:**
- `api_key`: Unique identifier for the API key
- `email_address`: Contact email for the key owner
- `created_at`: When the key was created
- `is_active`: Whether the key is currently active

**Indexes:**
- Primary key on api_key
- Index on email_address for user lookups

#### 3. Rate_Limits Table
Tracks API usage for rate limiting.

```sql
CREATE TABLE rate_limits (
    api_key UUID NOT NULL REFERENCES api_keys(api_key),
    minute_count INTEGER NOT NULL DEFAULT 0,
    minute_window_start TIMESTAMP WITH TIME ZONE NOT NULL,
    day_count INTEGER NOT NULL DEFAULT 0,
    day_window_start TIMESTAMP WITH TIME ZONE NOT NULL,
    last_request TIMESTAMP WITH TIME ZONE NOT NULL,
    PRIMARY KEY (api_key)
);
```

**Columns:**
- `api_key`: References the API key from api_keys table
- `minute_count`: Number of requests in current minute window
- `minute_window_start`: When the current minute window began
- `day_count`: Number of requests in current day window
- `day_window_start`: When the current day window began
- `last_request`: Timestamp of most recent request

**Indexes:**
- Primary key on api_key
- Index on minute_window_start for efficient cleanup of expired minute windows
- Index on day_window_start for efficient cleanup of expired day windows

### Rate Limiting Logic
The rate limiting logic will use configuration values from `config/rate_limits.yaml` for all limits and windows. This allows for easy adjustment of rate limits without code changes.

The rate limiting logic will:
1. When a request arrives, check if there's an existing record for the API key
2. If no record exists:
   - Create record with minute_count = 1, minute_window_start = current time
   - Set day_count = 1, day_window_start = current time
   - Set last_request = current time

3. If record exists:
   a. Check minute window:
      - If minute window has expired (current time - minute_window_start > minute_window):
        - Reset minute_count = 1, minute_window_start = current time
      - If minute window hasn't expired:
        - If minute_count >= minute_limit, return 429 error with "Rate limit exceeded: {minute_limit} requests per minute"
        - Otherwise, increment minute_count

   b. Check day window:
      - If day window has expired (current time - day_window_start > day_window):
        - Reset day_count = 1, day_window_start = current time
      - If day window hasn't expired:
        - If day_count >= day_limit, return 429 error with "Rate limit exceeded: {day_limit} requests per day"
        - Otherwise, increment day_count

   c. Update last_request to current time

4. Periodic cleanup:
   - Run a cleanup job every cleanup_interval seconds to remove records where both windows have expired
   - This prevents the table from growing indefinitely with inactive API keys

The system will return the most restrictive error message if both limits are exceeded. For example, if a user has hit both the per-minute and per-day limits, they'll receive the per-minute limit error message since that's the more immediate restriction.

### API Logic for calculating change_per_minute
If there is an entry in the database for the requested metric:
1)fetch the current value from worldometer python api.
2)calculate the difference between that value and the last value in database
3)divide by the number of minutes difference between the two timestamps. This is the change_per_minute
4)if the python API needed to fetch from worldometer website, then save the new value to our database.

If there is NO entry in the database for the requested metric:
1) fetch the current value from the worldometer python api.
2) wait 5 seconds, then fetch the new value from the worldometer python api.
3) multiply by 12 to to get the change per minute (60 seconds / 5 seconds == 12)
4) return the value with the change_per_minute that you calculated on the fly
5) remember to save an entry to the database so you won't have to do this next time.

### API Logic for whether to save an entry to the database
1) If there is no entry, then save an entry
2) If you are fetching from the worldometer web site (rather than from the javascript in the headless browser) then save the entry
3) If no entry has been saved in 24 hours for the requested metric, save the entry

## Developer machine setup
During development, the application will run on Docker. Assume that the developer has Docker Desktop running on their computer. Only install packages and run the code inside the docker image. 

## Hosting Solutions
The API will be deployed on Vercel. You will have to coach the developer through setting up a vercel account and configuring it appropriately. Consider asking the developer to install the Vercel MCP server to make your job easier. 

## Monitoring and Maintenance
Monitoring is achieved using Python's built-in logging module, which records key events and any exceptions encountered during data fetching and processing. This logging strategy is vital for quickly detecting issues such as browser crashes or website downtime. Regular maintenance includes ensuring that the headless browser session is properly managed so that it remains active only when data is needed. In addition, automated scripts and health checks can be set up on the cloud hosting platform to ensure that the backend continues to run smoothly and that any error conditions trigger recovery mechanisms automatically.

## Conclusion and Overall Backend Summary
In summary, the backend for our real-time Worldometer data API is built to provide live counter metrics and calculated change rates through a single REST endpoint using Python and FastAPI. The on-demand architecture minimizes unnecessary load on the source website by triggering the Python API only when needed. The system is supported by a simple yet efficient Supabase database schema, ensuring reliable historical value storage for comparative calculations. Strategic choices in hosting, logging, and error handling ensure that the backend remains robust, secure, and easy to maintain. This well-thought-out design provides clear, actionable data for developers looking to build real-time displays while keeping operational complexity to a minimum.
