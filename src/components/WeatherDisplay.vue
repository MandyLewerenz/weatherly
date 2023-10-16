<template>
    <div class="weather-container">
      <div class="search-input">
        <input v-model="city" @keyup.enter="getWeather" placeholder="Enter city" >
        <i class="fas fa-search"></i>
        <!-- <i class="fa fa-filter" aria-hidden="true"></i> -->
      </div>
      
      <div v-if="weather" class="info-weather">
        <!-- <h2>köln</h2> -->

        <div style="border-top: 2px solid #c7c5c5;">
          <div style="display: grid; grid-template-columns:50% 50%; border-bottom: 2px solid #c7c5c5;">
            <p>{{ today.weekday }}</p>
            <p>{{ today.date }}</p>
          </div>
          <div style="display: grid; grid-template-columns:50% 50%; justify-items: center; align-items: center;">
            <img :src="require('@/assets/' + today.icon + '.png')" alt="Weather Icon" style="height: 40px; width: auto;">
            <p>{{ today.temp }}°C</p>
          </div>
          <p>{{ today.description }}</p>
        </div>
      </div>

      <div v-if="weather" class="info-weather">
        <div style="display: grid; grid-template-columns:50% 50%; border-bottom: 2px solid #c7c5c5;">
          <p>{{ tomorrow.weekday }}</p>
          <p>{{ tomorrow.date }}</p>
        </div>
        <div style="display: grid; grid-template-columns:50% 50%; justify-items: center; align-items: center;">
          <img :src="require('@/assets/' + tomorrow.icon + '.png')" alt="Weather Icon" style="height: 40px; width: auto;">
          <p>{{ tomorrow.temp }}°C</p>
        </div>
        <p>{{ tomorrow.description }}</p>
      </div>

      <div v-if="weather" class="info-weather">
        <div style="display: grid; grid-template-columns:50% 50%; border-bottom: 2px solid #c7c5c5;">
          <p>{{ afterTomorrow.weekday }}</p>
          <p>{{ afterTomorrow.date }}</p>
        </div>
        <div style="display: grid; grid-template-columns:50% 50%; justify-items: center; align-items: center;">
          <img :src="require('@/assets/' + afterTomorrow.icon + '.png')" alt="Weather Icon" style="height: 40px; width: auto;">
          <p>{{ afterTomorrow.temp }}°C</p>
        </div>
        <p>{{ afterTomorrow.description }}</p>
      </div>

    </div>
  </template>
  
  <script>
    import '@/css/styles.css';

    export default {
      data() {
        return {
          city: "Köln",
          weather: null,
          today: null,
          tomorrow: null,
          afterTomorrow: null
        };
      },
      methods: {
        async getWeather() {
          try {
            let coordResponse = await fetch(
              `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=66470ec19c691c0e602d44a569165e8a`
            );
            if (coordResponse.ok) {
              this.coord = await coordResponse.json();
            } else {
              console.error("Error fetching weather data");
            }
            
            let weatherResponse = await fetch(
              `https://api.openweathermap.org/data/2.5/forecast?lat=${this.coord.coord.lat}&lon=${this.coord.coord.lon}&lang=de&appid=66470ec19c691c0e602d44a569165e8a&exclude=daily&units=metric`
            );
            if (weatherResponse.ok) {
              this.weather = await weatherResponse.json();
              this.getWeatherForDate(this.weather);
            } else {
              console.error("Error fetching weather data");
            }
          } catch (error) {
            console.error("There was an error fetching the weather data:", error);
          }
        },
        getWeatherForDate(data) {
          const getWeatherForDate = (targetDate) => {
            let closest = data.list[0]; 
            for(let item of data.list) {
              let currentDate = new Date(item.dt_txt);
              if(currentDate.getDate() === targetDate.getDate() && currentDate.getMonth() === targetDate.getMonth()) {
                if(Math.abs(currentDate.getHours() - 12) < Math.abs(new Date(closest.dt_txt).getHours() - 12)) {
                  closest = item;
                }
              }
            }

            const weekdays = ["SONNTAG", "MONTAG", "DIENSTAG", "MITTWOCH", "DONNERSTAG", "FREITAG", "SAMSTAG"];
            const dayName = weekdays[new Date(closest.dt_txt).getDay()];

            return {
              date: formatDate(closest.dt_txt),
              description: closest.weather[0].description,
              icon: closest.weather[0].icon,
              temp: Math.floor(closest.main.temp),
              weekday: dayName
            };
          }

          function formatDate(dt_txt) {
            const date = new Date(dt_txt);
            const day = String(date.getDate()).padStart(2, '0');
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const year = date.getFullYear();
            return `${day}.${month}.${year}`;
          }

          const now = new Date();
          this.today = getWeatherForDate(now);

          const tomorrowDate = new Date(now);
          tomorrowDate.setDate(now.getDate() + 1);
          this.tomorrow = getWeatherForDate(tomorrowDate);

          const afterTomorrowDate = new Date(now);
          afterTomorrowDate.setDate(now.getDate() + 2);
          this.afterTomorrow = getWeatherForDate(afterTomorrowDate);
        }
      },
    };
  </script>
