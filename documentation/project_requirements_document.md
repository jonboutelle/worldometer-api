# Project Requirements Document

## 1. Project Overview
This project aims to create a REST API that exposes the live data provided by the "Live Counters" section of the worldometer website. It does this by using an existing Python API that runs a headless browser to load and update the worldometer.info page in real time. The focus is on exposing critical metrics—such as current population, births today, births this year, deaths today, deaths this year, net population growth today, and net population growth this year—using a single REST endpoint. In addition to returning these live numbers, the API will calculate and include a "change_per_minute" metric that represents how fast these values are changing.

Developers will use this API to build real-time displays without having to constantly poll the Python API or the worldometer website (because they can increment using the change_per_minute attribute). The system is intentionally designed to work on a demand basis, where the Python API is invoked only when a request arrives. This project’s key objectives are to ensure minimal load on the source website by avoiding constant background polling, provide accurate current and rate-of-change data, and offer robust error handling that includes fallback data with an identifying flag when issues occur.

## 2. In-Scope vs. Out-of-Scope

**In-Scope:**

*   Implementation of a REST API that returns all live counter metrics and a computed change_per_minute for each metric.
*   Integration with the existing Python API that controls a headless browser loading worldometer.info.
*   Use of a Supabase relational database to store one historical record per metric per day with key, value, and UTC ISO8601 timestamp.
*   On-demand fetching, meaning the Python API is triggered only upon a REST API request.
*   Calculation of the change_per_minute by comparing the most recent stored value with the current value.
*   Comprehensive error and exception handling (including website downtime, headless browser crashes) with fallback data and logging using Python's logging module.

**Out-of-Scope:**

*   Implementing any functionality outside the "Live Counters" section of the worldometer API.
*   Creating additional endpoints for metrics not related to live counters.
*   Scheduled or periodic polling of data; the API will not update data unless a REST request is made.
*   Advanced scalability or complex caching, as high scalability is not a required objective.

## 3. User Flow
A developer initiates a REST API request to fetch live data from the worldometer source. Upon receiving the request, the API immediately triggers the Python API which is already running a headless browser session showing the worldometer.info page. The API retrieves the latest live counter metrics and then looks up the corresponding most recent historical record for each metric stored in the Supabase database. Using the historical record, it calculates the change_per_minute by comparing the current value with the last historical value, taking into account the time difference based on the UTC ISO8601 timestamps.

After processing the data, the API compiles a single response payload containing the requested metric along with the change_per_minute values. If the live data fetching fails at any point—whether due to a website issue or a headless browser crash—the API provides fallback data while marking the response with a flag to indicate that the data may not be real-time. All errors and events are logged using Python's logging module to facilitate future troubleshooting and to support any recovery actions within the Python API.

## 4. Core Features

*   **Single REST API Endpoint:**

    *   An endpoint that returns the requested live counter metric.
    *   Payload includes current values for metrics such as current_population, births_today, births_this_year, deaths_today, deaths_this_year, net_population_growth_today, and net_population_growth_this_year.

*   **On-Demand Data Fetching:**

    *   The Python API and headless browser are invoked only when a REST API call arrives.
    *   This design minimizes load on the worldometer website.

*   **Historical Data Storage:**

    *   Each metric’s historical value is stored in a Supabase database with a schema containing key, value, and timestamp (in UTC ISO8601 format).
    *   Only one historical record per key per day is maintained to facilitate change rate calculations.

*   **Change Rate Calculation:**

    *   For every API call, the current metric value is compared against the stored historical value to compute change_per_minute.

*   **Robust Error Handling and Fallback:**

    *   All exceptions from the Python API and headless browser events are trapped and logged.
    *   In situations where live data cannot be fetched, the API returns fallback data (both the value and the computed change rate) along with an indicator flag.

*   **Logging and Operational Resilience:**

    *   Implementation of Python’s logging module to capture errors, exceptions, and events within the data fetching and processing workflow.

## 5. Tech Stack & Tools

*   **Frontend/Client (API Consumption):**

    *   Not applicable directly as this is a REST API service; however, the API responses are crafted in standard JSON format for easy consumption.

*   **Backend Frameworks & Languages:**

    *   Python using the FastAPI framework to handle REST endpoints.
    *   Leveraging the Python worldometer API that utilizes a headless browser.

*   **Database:**

    *   Supabase is used to store historical data with a simple table schema that includes key, value, and timestamp in UTC ISO8601 format.

*   **Additional Libraries & Tools:**

    *   Python’s logging module for error logging and monitoring.
    *   Integration with Cursor, the IDE that provides AI-powered coding suggestions, to streamline development.

## 6. Non-Functional Requirements

*   **Performance:**

    *   The API should respond promptly, with target response times ideally within 500ms, given that the data is fetched on-demand from an already-running headless browser.

*   **Reliability & Resilience:**

    *   The system must gracefully handle errors from the underlying Python API, including website downtime and headless browser crashes.
    *   Fallback responses with clearly indicated flags are to be provided when errors occur.

*   **Security & Compliance:**

    *   Ensure secure handling of data transactions, even though the API is expected to be lightweight.
    *   Maintain proper logging of errors and events for auditing and troubleshooting, using Python’s logging module.

*   **Usability:**

    *   The JSON response from the single endpoint should be easy to parse and integrate into real-time displays.
    *   Documentation should make it clear how clients can differentiate between live and fallback data.

## 7. Constraints & Assumptions

*   **Dependencies:**

    *   The project depends on the availability and stability of the worldometer.info website and the reliable operation of the headless browser.
    *   Assumes that the Python worldometer API can keep a headless browser session open and fetch real-time updates.

*   **Assumptions:**

    *   Historical data storage is limited to one record per key per day, which is considered sufficient for calculating the change_per_minute.
    *   The system operates solely on-demand; no continuous or scheduled polling is executed unless triggered by an API call.
    *   Timestamps are consistently stored in UTC using ISO8601 format.
    *   Given the expected low usage, the design will not include advanced caching


## 8. Known Issues & Potential Pitfalls

*   **Website and Headless Browser Dependency:**

    *   If worldometer.info becomes inaccessible or the headless browser crashes, the API must rely on fallback data. Robust error handling and recovery strategies need to be in place to address these issues.

*   **Error Handling Complexity:**

    *   Capturing and handling all kinds of exceptions from the Python API (ranging from network issues to browser crashes) can be challenging.
    *   Use comprehensive logging with Python’s logging module to record and monitor these events and possibly implement retries or recovery strategies.

*   **Timing and Rate Calculation:**

    *   Ensuring that the calculated change_per_minute accurately represents real-time changes requires precise time comparisons between the stored historical value and the current value.
    *   It is important to ensure that the timestamp differences are computed correctly in seconds and then converted appropriately to a per-minute rate.

*   **Future Scalability Considerations:**

    *   Although scalability is not a current priority, any unexpected increase in API usage might require revisiting the design for caching or rate limiting strategies.

This document should serve as a clear and unambiguous guide for any further technical documents and development work on this project. The guidelines provided cover all critical aspects of the design and functionality expected from the REST API for real-time worldometer data integration.
