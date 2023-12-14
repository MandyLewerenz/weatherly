<template>
  <div class="container d-flex flex-column vh-100">
    <div class="row align-items-center justify-content-center flex-fill">

      <div class="col-10 col-md-4">

        <div class="search-input">
          <input v-model="city" @keyup.enter="getWeather" placeholder="Stadt eingeben">
          <i class="fas fa-search"></i>
        </div>

      </div>
    </div>
    <div v-if="weatherData.length" class="row justify-content-md-around justify-content-center flex-fill show-weather">
      <div v-for="weather in weatherData" :key="weather.id" class="col-10 col-md-3 ">
        <div class="info-weather px-4">

          <div class="row border-date">
            <div class="col-6 my-2 p-0 text-start">{{ weather.weekday }}</div>
            <div class="col-6 my-2 p-0 text-end">{{ weather.date }}</div>
          </div>

          <div class="row align-items-center justify-content-center">
            <div class="col-4 p-0 my-2">
              <div class="temp-icon"
                :class="{ 'temp-icon-day': weather.icon.includes('d'), 'temp-icon-night': weather.icon.includes('n') }">
                <img :src="require('@/assets/' + weather.icon + '.png')" alt="Weather Icon">
              </div>
            </div>

            <div class="col-8 p-0 my-2 text-center">
              <div v-if="weather.temp"> Aktuell: {{ weather.temp }}°C</div>
              <div>Min: {{ weather.min }}°C </div>
              <div>Max: {{ weather.max }}°C</div>
              <div class="mt-2">{{ weather.description }}</div>
            </div>
          </div>
        </div>

      </div>
    </div>

    <!-- <errorDisplay /> -->
    <div v-if="error" class="row justify-content-md-around justify-content-center flex-fill">
      <div class="col-10 col-md-4">
        <div class="info-weather px-4">
          <div class="row border-date">
            <div class="col-6 my-2 p-0">FEHLER</div>
          </div>

          <div class="row align-items-center">
            <div class="col-12 p-0 my-2">
              Stadt nicht gefunden!
            </div>
          </div>

        </div>
      </div>
    </div>

  </div>
</template>
  
<script>
import '@/css/styles.scss';

export default {
  data() {
    return {
      city: "",
      weatherData: [],
      error: false,
    };
  },
  methods: {
    async getWeather() {
      try {
        this.city = this.city.trim();
        let coordResponse = await fetch(
          process.env.VUE_APP_API_WEATHER + `?q=${this.city}&units=metric&appid=` + process.env.VUE_APP_API_KEY
        );
        if (coordResponse.ok) {
          this.coord = await coordResponse.json();
          this.error = false;

          let currentWeatherResponse = await fetch(
            process.env.VUE_APP_API_WEATHER + `?lat=${this.coord.coord.lat}&lon=${this.coord.coord.lon}&lang=de&exclude=daily&units=metric&appid=` + process.env.VUE_APP_API_KEY
          );
          let forecastResponse = await fetch(
            process.env.VUE_APP_API_FORECAST + `?lat=${this.coord.coord.lat}&lon=${this.coord.coord.lon}&lang=de&exclude=daily&units=metric&appid=` + process.env.VUE_APP_API_KEY
          );
          if (currentWeatherResponse.ok && forecastResponse.ok) {

            // Options for formatting the date
            const dateOptions = { weekday: 'long', timeZone: 'Europe/Berlin' };

            this.weatherData = [];
            this.error = false;

            const currentData = await currentWeatherResponse.json();
            this.weatherData.push(
              {
                date: this.formatDate(new Date()),
                description: currentData.weather[0].description,
                icon: currentData.weather[0].icon,
                temp: currentData.main.temp.toFixed(0),
                min: currentData.main.temp_min.toFixed(0),
                max: currentData.main.temp_max.toFixed(0),
                weekday: this.getWeekdayNameForDate(new Date(), dateOptions)
              }
            );

            const forecastData = await forecastResponse.json();
            console.log(forecastData);
            this.processForecastData(forecastData, dateOptions);
          } else {
            console.error("Error fetching weather data");
            this.weatherData = [];
            this.error = true;
          }
        } else {
          console.error("Error fetching weather data");
          this.weatherData = [];
          this.error = true;
        }
      } catch (error) {
        console.error("There was an error fetching the weather data:", error);
        this.error = true;
      }
    },
    processForecastData(apiWeatherData, dateOptions) {
      const currentDate = new Date();

      console.log(currentDate);

      const getWeatherDataForDaysAhead = (daysAhead) => {
        const date = new Date(currentDate);
        date.setDate(currentDate.getDate() + daysAhead);
        console.log('Date: ' + date);
        const weatherForDay = apiWeatherData.list.filter((item) => calculateDayDifference(new Date(item.dt_txt)) === daysAhead);
        const weatherInfo = getMiddleDayWeatherInfo(weatherForDay);
        const tempData = findMinMaxTemperatures(weatherForDay);

        return {
          date: this.formatDate(date),
          description: weatherInfo.description,
          icon: weatherInfo.icon,
          temp: null,
          min: tempData.minTemp.toFixed(0),
          max: tempData.maxTemp.toFixed(0),
          weekday: this.getWeekdayNameForDate(date, dateOptions)
        };
      }

      //Get weatherData for tommorow
      this.weatherData.push(getWeatherDataForDaysAhead(1));
      //Get weatherData for the day after tommorow
      this.weatherData.push(getWeatherDataForDaysAhead(2));

      console.log(this.weatherData);

      // Function to calculate the day difference for an item to the currentDate
      function calculateDayDifference(itemDate) {

        const itemDateCopy = new Date(itemDate);
        itemDateCopy.setHours(0, 0, 0, 0); // Set hours, minutes, seconds, and milliseconds to 0

        const currentDateCopy = new Date(currentDate);
        currentDateCopy.setHours(0, 0, 0, 0); // Set hours, minutes, seconds, and milliseconds to 0

        const daysDifference = Math.floor((itemDateCopy - currentDateCopy) / (24 * 60 * 60 * 1000));
        console.log("Daydifference: " + daysDifference);
        return daysDifference;
      }

      function getMiddleDayWeatherInfo(weatherList) {
        const middleIndex = Math.floor(weatherList.length / 2);

        if (weatherList[middleIndex]) {
          // Extract the description and icon from the middle weather data
          const description = weatherList[middleIndex].weather[0].description;
          const icon = weatherList[middleIndex].weather[0].icon;
          return { description, icon };
        } else {
          return { description: 'No data', icon: 'na' };
        }
      }

      function findMinMaxTemperatures(weatherList) {
        if (!weatherList || weatherList.length === 0) {
          return { minTemp: null, maxTemp: null };
        }

        const { minTemp, maxTemp } = weatherList.reduce(
          (acc, item) => ({
            minTemp: Math.min(acc.minTemp, item.main.temp),
            maxTemp: Math.max(acc.maxTemp, item.main.temp),
          }),
          { minTemp: Infinity, maxTemp: -Infinity }
        );
        return { minTemp, maxTemp };
      }
    },

    // Get the German weekday name for the date
    getWeekdayNameForDate(date, dateOptions) {
      return date.toLocaleDateString('de-DE', dateOptions);
    },
    formatDate(date) {
      return date.toLocaleDateString('de-DE', { day: '2-digit', month: '2-digit', year: 'numeric' });
    }

  },
};
</script>
