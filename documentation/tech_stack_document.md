# Tech Stack Document

## Introduction

This project is designed to create a REST API that exposes real-time world data extracted from the live counters on the Worldometer website. By leveraging an existing Python API that uses a headless browser to load and monitor the worldometer page, the project provides up-to-date metrics such as current population figures, births today, births this year, deaths today, deaths this year, and net population growth figures. In addition to the live counters, the API calculates a "change per minute" value by comparing the latest data with a previously stored historical record. The system is built to be responsive only when a request comes in, thereby preventing unnecessary load on the Worldometer website and ensuring that the data is delivered on-demand and efficiently.

## Frontend Technologies

Although this project does not include a traditional frontend, its REST API is designed for easy consumption by client applications. All responses are returned in well-structured JSON format, making it simple for developers and non-technical users to integrate and display the data. The design of the API response is crafted to be developer-friendly, ensuring that the information including current values and the calculated change per minute is presented clearly without the need for complicated transformations on the client side.

## Backend Technologies

The backbone of the project is powered by Python, using the FastAPI framework to create the REST endpoints. FastAPI was chosen for its speed, ease of use, and excellent support for asynchronous operations, which is important for a system that relies on on-demand data fetching. The Python API makes use of a headless browser mechanism to interact with the Worldometer website and continuously update live data on a kept-open browser session. For data storage, Supabase is used as the database solution to store one historical data record per metric. The database schema is very straightforward, containing a key for each metric, the corresponding numeric value, and a timestamp in UTC ISO8601 format, ensuring time calculations such as change per minute are accurate and consistent.

## Infrastructure and Deployment

The system is set up to deploy on a reliable cloud hosting platform suitable for running Python applications with FastAPI. Although the project is not expected to handle high volumes of traffic, the choice of infrastructure ensures that the deployment remains scalable and cost effective if usage increases. The deployment process includes a CI/CD pipeline to allow for continuous integration and delivery, enabling smooth updates and quick bug fixes. Additionally, version control is maintained through a Git-based system, providing a robust framework for tracking changes and collaborating on the project over time.

## Third-Party Integrations

This project relies on several key third-party technologies and tools. The Python API leverages the official Worldometer library to interact with the live counters section of the Worldometer website. In addition, integration with Supabase provides a managed database solution designed for quick data retrieval and storage with the use of a simple table schema. Logging is implemented using Python’s built-in logging module, ensuring that any issues with the headless browser or data fetching process are captured for troubleshooting. Moreover, integration with Cursor, an advanced IDE providing AI-powered coding suggestions, has been included to streamline development and maintain high code quality throughout the development cycle.

## Security and Performance Considerations

For security, all timestamps are stored in a universally accepted UTC ISO8601 format to ensure consistency across distributed systems. The REST API includes robust error handling strategies, trapping all exceptions originating from the headless browser or the Python API. When errors occur, the API not only logs these events using Python’s logging module but also returns fallback data with an indication flag, ensuring that users are always provided with a meaningful response. Performance is optimized by using on-demand data fetching, meaning the underlying system activates only when a request is made, thereby reducing unnecessary load on the Worldometer website. The headless browser setup ensures that the number of requests to the source is kept minimal, making sure that performance remains stable even in the face of unexpected issues.

## Conclusion and Overall Tech Stack Summary

In summary, the technology choices in this project have been carefully selected to meet the need for a real-time data display service by using an on-demand architecture. Python with the FastAPI framework provides a robust and efficient backend, while Supabase offers a simple yet effective database for storing historical values in UTC ISO8601 format. The integration of the Worldometer Python API using a headless browser ensures live counters are updated in real time and combined with a calculated change per minute metric to offer meaningful insights. The inclusion of comprehensive logging, on-demand operation, and fallback strategies ensures that the system remains reliable and user-friendly even in unexpected situations. This tech stack has been designed with clarity, performance, and resilience in mind, making it a clear choice for developers looking to build real-time displays with minimal overhead and maximum data accuracy.
