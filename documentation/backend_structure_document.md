# Backend Structure Document

## Introduction

This document explains how the backend for our real-time Worldometer data API is set up. The purpose of this backend is to expose live data from the Worldometer website through a single REST API endpoint. The API not only provides the latest figures such as current population, births, deaths, and net population changes but also calculates how fast these values are changing by comparing them to previously stored records. Every part of the system—from the way the live data is fetched using a headless browser to the way historical data is stored and compared—has been designed to work only when a request is made. This reduces unnecessary load on the Worldometer website and keeps the system responsive and efficient.

## Backend Architecture

At the heart of the system is a Python-based backend powered by the FastAPI framework. The architecture is built around an on-demand data retrieval model, where the Python API, running a headless browser session, is triggered only when a client makes a REST API call. This design ensures that the backend remains lightweight and only performs operations when needed. The FastAPI framework was chosen for its simplicity and high performance in handling asynchronous tasks, which is ideal for fetching real-time data without continuous polling. The architecture emphasizes maintainability as each component has a clearly defined role, and scalability is considered by isolating on-demand operations to prevent unnecessary load.

## Database Management

The system uses Supabase as its database management solution. The database is structured with a straightforward schema that contains a key (such as the name of a metric like US_POPULATION), a value (the corresponding number), and a timestamp stored in UTC using ISO8601 format. This simple arrangement is enough to calculate the change per minute by comparing the current value with the stored historical value. By keeping only one record per key, the design minimizes data storage overhead while providing the necessary historical comparison for a smooth calculation of how quickly the values are changing.

## API Design and Endpoints

The API is designed as a single REST endpoint that returns all live counter metrics along with the computed change per minute. This endpoint is built using FastAPI and ensures that whenever a call is made, the system will trigger the headless browser session to fetch the latest data from the worldometer.info page. Once the current data is obtained, the API retrieves the most recent stored value from the Supabase database to calculate the change per minute. The response is structured in a clear JSON format, containing both the current metrics and an indicator flag that signifies whether the data returned is live or fallback data. This unified endpoint simplifies integration for developers who need real-time display data.

## Hosting Solutions

The backend is deployed on a cloud hosting platform that supports Python applications and FastAPI. The chosen hosting environment is designed for reliability, cost-effectiveness, and ease of scaling if necessary. Since our API only activates on-demand and is not expected to handle high volumes of traffic, the infrastructure remains lightweight. The setup ensures that the server is available to handle REST API requests quickly, while the cloud platform provides robust support for future scalability and maintenance without requiring significant additional resources.

## Infrastructure Components

Several infrastructure components work together to ensure the backend performs optimally. The system relies on a headless browser which continuously keeps the Worldometer page open to fetch live counters. This headless browser acts as a unique data source, ensuring that updates are received in near real-time without needing constant new sessions. Complementing this is the Supabase database which efficiently stores the historical data necessary for change rate calculations. Moreover, Python’s built-in logging module is integrated to capture any exceptions or operational anomalies. Each request passes through components that perform error handling, data fetching, comparison, and response compilation, ensuring a seamless interaction between the frontend and the backend services.

## Security Measures

Security in the backend largely focuses on ensuring the integrity and reliability of the data as it moves through different components. To start, all sensitive operations, such as data fetching from the Worldometer website, are wrapped in robust exception-handling constructs. Furthermore, whenever an error occurs—whether from the headless browser crashing or the website being down—the system defaults to providing fallback data, clearly marked with a flag in the response. Although advanced authentication protocols are not a priority given the backend exports public data, the system emphasizes secure handling of the data and precise logging. Storing timestamps in the universally accepted UTC ISO8601 format also helps maintain consistency across distributed systems.

## Monitoring and Maintenance

Monitoring is achieved using Python’s built-in logging module, which records key events and any exceptions encountered during data fetching and processing. This logging strategy is vital for quickly detecting issues such as browser crashes or website downtime. Regular maintenance includes ensuring that the headless browser session is properly managed so that it remains active only when data is needed. In addition, automated scripts and health checks can be set up on the cloud hosting platform to ensure that the backend continues to run smoothly and that any error conditions trigger recovery mechanisms automatically.

## Conclusion and Overall Backend Summary

In summary, the backend for our real-time Worldometer data API is built to provide live counter metrics and calculated change rates through a single REST endpoint using Python and FastAPI. The on-demand architecture minimizes unnecessary load on the source website by triggering the Python API only when needed. The system is supported by a simple yet efficient Supabase database schema, ensuring reliable historical value storage for comparative calculations. Strategic choices in hosting, logging, and error handling ensure that the backend remains robust, secure, and easy to maintain. This well-thought-out design provides clear, actionable data for developers looking to build real-time displays while keeping operational complexity to a minimum.
