# Weather App 
<div align="center">
    <img src="https://github.com/MandyLewerenz/portfolio/blob/master/src/assets/appLogoCloud.png" width="300">
</div>

## Table of Contents
* [Introduction](#introduction)
* [Technologies](#technologies)
* [Setup](#setup)
* [Screenshots](#screenshots)
  
## Introduction 
Weatherly is a Vue.js-based german weather forecasting application that provides current and 3-day weather data for any searched city.
It uses the OpenWeatherMap API, specifically the `/weather` and `/forecast` endpoints, for fetching accurate, real-time weather data. 

The decision to use OpenWeatherMap API's `/weather` and `/forecast` endpoints was motivated by their free and unrestricted access, unlike other APIs that have limitations.
Although the API doesn't directly provide forecasts based on city names and requires coordinates, this challenge was solved within the app.

The app's responsive design is crafted with the latest Bootstrap 5 and customized with SCSS. 
Unique weather icons, representing various weather conditions, are designed using Midjourney and Photoshop, adding a personal and creative touch to the app.

## Technologies
- Vue.js 3
- Bootstrap 5
- Sass 1.69
- OpenWeatherMap API 2.5

## Setup
1. Clone the repository.
2. Install dependencies:

```
npm install
```
3. Start the development server:
```
npm run serve
```
4. Open [http://localhost:8080/](http://localhost:8080/) in your browser.

## Screenshots
### City Search
<div align="left">
    <img src="https://github.com/MandyLewerenz/weatherly/assets/95027740/3fd9e903-af15-49f2-a1fa-c25f86f5288f" width="25%">
    <img src="https://github.com/MandyLewerenz/weatherly/assets/95027740/ad287457-7aa8-4fe5-a748-33eb61aeb899" width="70%">
</div>

### Forecast 
<div align="left">
    <img src="https://github.com/MandyLewerenz/weatherly/assets/95027740/d7898a63-c1f1-4832-80ae-67537b13bccf"  width="25%">
    <img src="https://github.com/MandyLewerenz/weatherly/assets/95027740/484058aa-0ce4-4775-9160-9c7b032eafe8" width="70%">
</div>
