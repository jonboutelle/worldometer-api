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

4.  **Configure Environment Variables**

    *   Action: Create a `.env` file at the project root to store environment variables (e.g., `SUPABASE_URL`, `SUPABASE_KEY`).
    *   File: `/worldometer_api/.env`
    *   Reference: PRD Section 5

5.  **Install Required Python Packages**

    *   Action: Install packages: FastAPI, uvicorn, worldometer (the Python API), supabase-py, and python-dotenv using pip:

    `pip install fastapi uvicorn worldometer supabase python-dotenv`

    *   Reference: Tech Stack

### Phase 2: Frontend Development

1.  **Frontend Not Applicable**

    *   Action: Since this project provides a REST API, no frontend component is required.
    *   Reference: PRD Section 1

### Phase 3: Backend Development

1.  **Create FastAPI Application Entry Point**

    *   Action: Create a file named `/app/main.py` to initialize a FastAPI application.
    *   Reference: PRD Section 1

2.  **Configure Logging in FastAPI**

    *   Action: In `/app/main.py`, import the `logging` module, configure the root logger, and set logging levels to record events and errors.
    *   Reference: PRD Section 4

3.  **Create Router for Live Counters Endpoint**

    *   Action: Create a new directory `/app/routers` and add the file `live_counters.py` to hold the REST endpoint logic for live counters.
    *   Reference: PRD Section 3

4.  **Import Worldometer Python API in Router**

    *   Action: In `/app/routers/live_counters.py`, add imports for the worldometer API so that live counter data can be fetched from the headless browser session.
    *   Reference: PRD Section 2

5.  **Implement REST Endpoint for Live Counters**

    *   Action: In `/app/routers/live_counters.py`, implement an async GET endpoint (e.g., `/api/v1/live-counters`) that performs the following:

        *   a. Call the worldometer API to fetch current live counter values.
        *   b. Retrieve the most recent historical record for each metric from the Supabase database.
        *   c. Calculate `change_per_minute` for each metric by comparing the current value with the historical value using the difference in UTC ISO8601 timestamps.

    *   Reference: Implementation Outline – Data Fetching and Historical Comparison

6.  **Implement Robust Error Handling and Fallback Strategy**

    *   Action: Wrap the live data fetch in a try/except block to catch exceptions (e.g., website downtime, headless browser crash). When an error occurs, log the error and set a fallback flag; use stored historical data for both the current value and change rate.
    *   Reference: PRD Section 4

7.  **Update Supabase with Current Values**

    *   Action: Upon a successful live data fetch, update the corresponding record in the Supabase database for each metric (overwriting the single historical record). Store the value along with the current UTC ISO8601 timestamp.
    *   Reference: PRD Section 3

8.  **Create Supabase Client Module**

    *   Action: Create a new directory `/app/db` and add a file named `supabase_client.py` to manage database connections and queries (both retrieval and update operations) with Supabase.
    *   Reference: PRD Section 3

9.  **Implement Database Query Functions**

    *   Action: In `/app/db/supabase_client.py`, write functions to:

        *   Connect to Supabase using the environment variables.
        *   Retrieve the latest historical record by key.
        *   Update the record for each metric.

    *   Reference: PRD Section 3

10. **Validate Change Rate Calculation**

    *   Action: Add logging statements in the endpoint to record the time difference and value difference used for computing `change_per_minute` to ensure accurate calculations.
    *   Reference: Implementation Outline – Data Storage and Change Rate Calculation

11. **Integrate the Router in the Main App**

    *   Action: Modify `/app/main.py` to include the live counters router, e.g., using `app.include_router(live_counters_router)`.
    *   Reference: Tech Stack – Backend Frameworks

12. **Optional: Add CORS Middleware (If Needed)**

    *   Action: In `/app/main.py`, add CORS middleware configuration if clients accessing the API require it.
    *   Reference: PRD Section 7

### Phase 4: Integration

1.  **Run FastAPI Application Locally**

    *   Action: Start the FastAPI service locally using uvicorn with the command:

    `uvicorn app.main:app --reload`

    *   Reference: Tech Stack – FastAPI

2.  **Perform Endpoint Testing using cURL**

    *   Action: Execute a GET request to `http://localhost:8000/api/v1/live-counters` using cURL or httpie to ensure that the response JSON includes all live counter metrics along with the computed `change_per_minute` and fallback flag when applicable.
    *   Command Example: `curl -X GET http://localhost:8000/api/v1/live-counters`
    *   Reference: PRD Section 4

3.  **Validate Supabase Database Operations**

    *   Action: After a successful API call, check the Supabase database table to verify that each metric record is updated with the latest value and timestamp.
    *   Reference: PRD Section 3

### Phase 5: Deployment

1.  **Create a Dockerfile for Containerization**

    *   Action: At the project root, create a `Dockerfile` that:

        *   Uses an official Python base image.
        *   Copies the project files into the container.
        *   Installs dependencies from `requirements.txt` (generate this file from pip freeze if desired).
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

4.  **Document Deployment Process for Cloud**

    *   Action: Create a deployment guide outlining steps to deploy the containerized FastAPI app (e.g., on Heroku, AWS, or any preferred platform), ensuring that environment variables (like `SUPABASE_URL` and `SUPABASE_KEY`) are set up in the cloud environment.
    *   Reference: PRD Section 7

5.  **Final End-to-End Validation**

    *   Action: Deploy to a staging environment and perform an end-to-end test by making GET requests to the deployed API endpoint to confirm that live data fetching, database updates, error handling, and fallback responses work as expected.
    *   Reference: Q&A – Pre-Launch Checklist

This plan ensures that the system only invokes the Python API on demand, minimizes the load on the worldometer website by maintaining a single headless browser session, provides accurate calculation of change rates by comparing with a historical record stored in Supabase, and properly logs and handles any errors that occur.

*Note: All development work should be executed in Cursor, our AI-powered IDE, to benefit from real-time code suggestions and validations.*
