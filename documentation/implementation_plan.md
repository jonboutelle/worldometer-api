## Implementation Plan for Worldometer Live Counters REST API

This plan outlines the steps to build a REST API using Python, FastAPI, and Supabase that exposes the live counter metrics from worldometer.info via the worldometer Python API. The API will calculate a change_per_minute for each metric by comparing current live data with a stored historical value. The system will handle errors robustly and provide fallback data as needed.

### Phase 1: Environment Setup

1.  **Create Project Directory**

    *   Action: Create a folder named `worldometer_api` for the project.
    *   Location: Root project directory.
    *   Reference: PRD Section 1

2.  **Initialize Git Repository**

    *   Action: Initialize a Git repository inside `worldometer_api` and create `main` and `dev` branches with appropriate branch protection rules.
    *   Reference: PRD Section 1

3.  **Set Up Python Virtual Environment**

    1.  Use Docker to set up an environment for this project. Assume the machine the developer is working on has Docker Desktop installed.
    2.  Create a `Dockerfile` for development environment
    3.  Create a `docker-compose.yml` for easy development setup

4.  **Configure Environment Variables**

    *   Action: Create a `.env` file at the project root to store environment variables (e.g., `SUPABASE_URL`, `SUPABASE_KEY`).
    *   File: `/worldometer_api/.env`
    *   Reference: PRD Section 5

5.  **Install Required Python Packages**

    *   Action: Create `requirements.txt` with the following packages:
        - fastapi==0.109.2
        - uvicorn==0.27.1
        - python-dotenv==1.0.1
        - supabase==2.3.4
        - pydantic==2.6.1
        - python-multipart==0.0.9
    *   Reference: Tech Stack

### Phase 2: Database Setup

1.  **Create SQL Schema**

    *   Action: Create SQL files to define the database schema:
        - `metrics` table for storing historical values
        - `api_keys` table for API key management
        - `rate_limits` table for rate limiting with per-minute and per-day limits
    *   File: `/sql/schema.sql`
    *   Reference: Backend Structure Document

2.  **Set Up Supabase Tables**

    *   Action: Execute the SQL schema in Supabase to create the required tables
    *   Reference: Backend Structure Document

### Phase 3: Python API Integration

1.  **Test Headless Browser Functionality**

    *   Action: Create tests to verify the headless browser behavior of the worldometer Python API:
        - Verify that subsequent calls use cached data from the browser
        - Document when the browser makes new requests to worldometer.info
        - Test browser session management
    *   File: `/tests/test_headless_browser.py`
    *   Reference: Backend Structure Document

2.  **Create FastAPI Application Entry Point**

    *   Action: Create a file named `/app/main.py` to initialize a FastAPI application.
    *   Reference: PRD Section 1

3.  **Configure Logging in FastAPI**

    *   Action: In `/app/main.py`, import the `logging` module, configure the root logger, and set logging levels to record events and errors.
    *   Reference: PRD Section 4

4.  **Create Router for Metrics Endpoint**

    *   Action: Create a new directory `/app/routers` and add the file `metrics.py` to hold the REST endpoint logic for metrics.
    *   Reference: PRD Section 3

5.  **Import Worldometer Python API in Router**

    *   Action: In `/app/routers/metrics.py`, add imports for the worldometer API so that live counter data can be fetched from the headless browser session.
    *   Reference: PRD Section 2

6.  **Implement REST Endpoint for Metrics**

    *   Action: In `/app/routers/metrics.py`, implement an async GET endpoint (e.g., `/api/v1/metrics/{metric_key}`) that performs the following:

        *   a. Validate the metric_key against the list of available metrics
        *   b. Call the worldometer API to fetch current live counter value
        *   c. Retrieve the most recent historical record from the Supabase database
        *   d. Calculate `change_per_minute` by comparing current and historical values
        *   e. Return the metric data with change rate and fallback flag

    *   Reference: Implementation Outline – Data Fetching and Historical Comparison

7.  **Implement Rate Limiting**

    *   Action: Create rate limiting middleware that:
        - Reads configuration from `config/rate_limits.yaml`
        - Enforces per-minute and per-day limits
        - Handles window expiration and cleanup
        - Returns appropriate 429 errors with limit information
    *   File: `/app/middleware/rate_limit.py`
    *   Reference: Backend Structure Document

8.  **Implement Robust Error Handling and Fallback Strategy**

    *   Action: Wrap the live data fetch in a try/except block to catch exceptions (e.g., website downtime, headless browser crash). When an error occurs, log the error and set a fallback flag; use stored historical data for both the current value and change rate.
    *   Reference: PRD Section 4

9.  **Update Supabase with Current Values**

    *   Action: Upon a successful live data fetch, update the corresponding record in the Supabase database for the metric (overwriting the single historical record). Store the value along with the current UTC ISO8601 timestamp.
    *   Reference: PRD Section 3

10. **Create Supabase Client Module**

    *   Action: Create a new directory `/app/db` and add a file named `supabase_client.py` to manage database connections and queries (both retrieval and update operations) with Supabase.
    *   Reference: PRD Section 3

11. **Implement Database Query Functions**

    *   Action: In `/app/db/supabase_client.py`, write functions to:

        *   Connect to Supabase using the environment variables.
        *   Retrieve the latest historical record by key.
        *   Update the record for each metric.
        *   Handle rate limit tracking and cleanup.

    *   Reference: PRD Section 3

12. **Validate Change Rate Calculation**

    *   Action: Add logging statements in the endpoint to record the time difference and value difference used for computing `change_per_minute` to ensure accurate calculations.
    *   Reference: Implementation Outline – Data Storage and Change Rate Calculation

13. **Integrate the Router in the Main App**

    *   Action: Modify `/app/main.py` to include the metrics router, e.g., using `app.include_router(metrics_router)`.
    *   Reference: Tech Stack – Backend Frameworks

14. **Add API Key Authentication**

    *   Action: Create authentication middleware that:
        - Validates API keys against the database
        - Checks if keys are active
        - Adds key information to request state
    *   File: `/app/middleware/auth.py`
    *   Reference: Backend Structure Document

### Phase 4: Integration

1.  **Run FastAPI Application Locally**

    *   Action: Start the FastAPI service locally using uvicorn with the command:

    `uvicorn app.main:app --reload`

    *   Reference: Tech Stack – FastAPI

2.  **Perform Endpoint Testing using cURL**

    *   Action: Execute GET requests to test various scenarios:
        - Valid metric key with API key
        - Invalid metric key
        - Invalid/missing API key
        - Rate limit exceeded
    *   Command Examples:
        ```bash
        curl -X GET http://localhost:8000/api/v1/metrics/WORLD_POPULATION -H "X-API-Key: your-api-key"
        ```
    *   Reference: PRD Section 4

3.  **Validate Supabase Database Operations**

    *   Action: After successful API calls, check the Supabase database tables to verify:
        - Metric values are updated correctly
        - Rate limits are tracked properly
        - API key validation works
    *   Reference: PRD Section 3

### Phase 5: Deployment

1.  **Create a Dockerfile for Containerization**

    *   Action: At the project root, create a `Dockerfile` that:

        *   Uses an official Python base image.
        *   Copies the project files into the container.
        *   Installs dependencies from `requirements.txt`.
        *   Sets the command to run uvicorn to start the FastAPI app.

    *   File: `/Dockerfile`

    *   Reference: Tech Stack – Deployment

2.  **Build the Docker Image**

    *   Action: Run the command `docker build -t worldometer_api .` to build the Docker image.
    *   Reference: Tech Stack – Deployment

3.  **Run the Docker Container Locally**

    *   Action: Run the built Docker image locally using:

    `docker run -p 8000:8000 --env-file .env worldometer_api`

    *   Reference: Tech Stack – Deployment

4.  **Set Up Vercel Deployment**

    *   Action: Create Vercel configuration files:
        - `vercel.json` for project settings
        - Environment variables in Vercel dashboard
    *   Reference: Backend Structure Document

5.  **Install and Configure Vercel MCP Server**

    *   Action: Guide the developer through:
        - Installing Vercel MCP server
        - Setting up authentication
        - Configuring deployment settings
    *   Reference: Backend Structure Document

6.  **Document Deployment Process for Cloud**

    *   Action: Create a deployment guide outlining steps to deploy the containerized FastAPI app on Vercel, ensuring that environment variables (like `SUPABASE_URL` and `SUPABASE_KEY`) are set up in the cloud environment.
    *   Reference: PRD Section 7

7.  **Final End-to-End Validation**

    *   Action: Deploy to a staging environment and perform an end-to-end test by making GET requests to the deployed API endpoint to confirm that:
        - Live data fetching works
        - Database updates are correct
        - Error handling and fallback responses work
        - Rate limiting functions properly
        - API key authentication works
    *   Reference: Q&A – Pre-Launch Checklist

This plan ensures that the system only invokes the Python API on demand, minimizes the load on the worldometer website by maintaining a single headless browser session, provides accurate calculation of change rates by comparing with a historical record stored in Supabase, and properly logs and handles any errors that occur.

*Note: All development work should be executed in Cursor, our AI-powered IDE, to benefit from real-time code suggestions and validations.*
