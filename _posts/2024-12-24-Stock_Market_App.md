---
layout: post
title: Stock Market Tracker
date: 24-12-2024
categories: [Applications]
tags: [data_analysis]
---
Investing in the stock market can be very rewarding and challenging. Effective tracking and analysis are important for making informed decisions. To help with this process, I am developing a C++ application designed to monitor real-time stock data, visualize performance metrics, and provide insightful data analytics through comprehensive graphs. This project shows my expertise in web scraping, data parsing, GUI development, and data visualization using modern C++ technologies.

<!-- ![Introduction Image](path-to-your-image/introduction.png) -->

## Project Components

The application is structured into four primary components:

1. **Web Scraper**: Utilizes `libcurl` to fetch real-time stock data from Yahoo Finance.
2. **CSV Parser**: Processes the retrieved CSV data to extract essential stock information.
3. **Stock Selector Inputs**: Provides a user interface for selecting and managing the stocks to track.
4. **Graphical User Interface (GUI)**: Displays the data using interactive and informative graphs.

<!-- ![Components Overview](path-to-your-image/components.png) -->

## Technical Implementation

### Framework Setup

The development began with the **Stalkerized TI and GUI framework**, which I chose for its simplicity and extendability. After cloning and setting up the framework, I prepared the main branch for integrating new features.

<!-- ![Framework Setup](path-to-your-image/framework-setup.png) -->

### Building the Default Application

To ensure the framework was correctly set up, I built and ran the default application, which provided a foundational GUI for further development.

```bash
# Build the application
build

# Run the application
run
```

<!-- ![Default App](path-to-your-image/default-app.png) -->

### Integrating Dependencies

For web scraping capabilities, `libcurl` and SSL libraries were added to the Dockerfile. This enabled the application to handle secure HTTPS requests essential for fetching data from Yahoo Finance.

```dockerfile
# Dockerfile snippet
RUN apt-get update && apt-get install -y curl libssl-dev
```

<!-- ![Docker Configuration](path-to-your-image/docker-config.png) -->

### Developing the Web Scraper

A custom curl wrapper was implemented in C++ to manage HTTP requests and retrieve data from Yahoo Finance. This wrapper abstracts the complexity of handling raw HTTP transactions, providing a more object-oriented interface for data retrieval.

<!-- ![Web Scraper Code](path-to-your-image/web-scraper-code.png) -->

### Implementing the CSV Parser

The CSV parser was designed to extract critical stock data, including dates, open, high, low, and close values. Utilizing regular expressions, the parser efficiently processes each line of the CSV file, ensuring accurate data extraction and storage.

<!-- ![CSV Parser Code](path-to-your-image/csv-parser-code.png) -->

### Designing the Graphical User Interface

Leveraging ImGui, I developed an intuitive user interface that allows users to input stock symbols, select date ranges, and visualize data through candlestick charts. The GUI ensures a seamless and interactive user experience.

<!-- ![GUI Design](path-to-your-image/gui-design.png) -->

## Testing and Validation

To ensure the application's reliability and accuracy, comprehensive unit tests were implemented using the Catch2 framework. These tests validate critical functionalities, including URL generation and data retrieval processes.

```cpp
// Example test case
TEST_CASE("URL Generation") {
    CurlWrapper curl;
    REQUIRE(curl.generateURL("AAPL", startDate, endDate) == expectedURL);
}
```

<!-- ![Testing](path-to-your-image/testing.png) -->

## Data Visualization

The application features candlestick charts to represent stock performance. Green candles indicate a rise in stock price, while red candles signify a decline. Custom plotting was achieved by extending the ImGui plot library to support the desired chart types, ensuring accurate and visually appealing data representation.

<!-- ![Candlestick Chart](path-to-your-image/candlestick-chart.png) -->

### Custom Plotting Enhancements

Since the ImGui plot library did not natively support candlestick charts, I extended its functionality to accommodate custom plotting requirements. This involved creating custom blocks and integrating them seamlessly with the existing plotting mechanisms.

<!-- ![Custom Plotting](path-to-your-image/custom-plotting.png) -->

## Running the Application

After completing the development and testing phases, the application is ready for deployment. Users can input stock symbols and date ranges to fetch and visualize corresponding data in real-time, providing a comprehensive dashboard for monitoring stock performance.

```bash
# Build and run
build
run
```

<!-- ![Running Application](path-to-your-image/running-application.png) -->

## Conclusion

The Stock Market Tracker Application exemplifies the integration of various technologies to create a functional and user-friendly tool for monitoring stock performance. By combining web scraping, data parsing, GUI development, and data visualization, the application provides valuable insights and enhances the investment tracking experience.

<!-- ![Conclusion Image](path-to-your-image/conclusion.png) -->