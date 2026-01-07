<!DOCTYPE html><html lang="en"><head>  <meta charset="UTF-8">  <meta name="viewport" content="width=device-width, initial-scale=1.0">  <title>Dynamic Weather Information System</title>  <style>
  
    body {
  
      font-family: Arial, sans-serif;
  
      background: #f0f2f5;
  
      margin: 0;
  
      padding: 0;
  
    }
  
    .container {
  
      max-width: 900px;
  
      margin: 20px auto;
  
      background: #fff;
  
      padding: 20px;
  
      border-radius: 12px;
  
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  
    }
  
    h1 {
  
      text-align: center;
  
      color: #333;
  
    }
  
    #search-box {
  
      display: flex;
  
      justify-content: center;
  
      flex-wrap: wrap;
  
      margin: 20px 0;
  
      gap: 10px;
  
    }
  
    #city-input {
  
      padding: 10px;
  
      width: 60%;
  
      border: 1px solid #ccc;
  
      border-radius: 8px;
  
      font-size: 16px;
  
    }
  
    button {
  
      padding: 10px 16px;
  
      background: #007BFF;
  
      color: white;
  
      border: none;
  
      border-radius: 8px;
  
      cursor: pointer;
  
    }
  
    button:hover {
  
      background: #0056b3;
  
    }
  
    #history {
  
      margin: 15px 0;
  
      padding: 10px;
  
      background: #f8f9fa;
  
      border-radius: 10px;
  
    }
  
    .history-btn {
  
      margin: 5px;
  
      padding: 6px 12px;
  
      border-radius: 6px;
  
      background: #e3f2fd;
  
      cursor: pointer;
  
      border: 1px solid #90caf9;
  
    }
  
    .clear-btn {
  
      margin-left: 10px;
  
      background: #dc3545;
  
    }
  
    #weather-result {
  
      text-align: center;
  
      margin-top: 20px;
  
    }
  
    #weather-icon {
  
      width: 100px;
  
      height: 100px;
  
    }
  
    .forecast-container {
  
      display: flex;
  
      justify-content: space-between;
  
      margin-top: 20px;
  
      flex-wrap: wrap;
  
    }
  
    .forecast-card {
  
      background: #f8f9fa;
  
      border-radius: 10px;
  
      padding: 10px;
  
      flex: 1 1 120px;
  
      margin: 5px;
  
      text-align: center;
  
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
  
    }
  
    .aqi-info {
  
      margin-top: 10px;
  
      padding: 5px;
  
      border-radius: 8px;
  
      color: #fff;
  
      font-weight: bold;
  
    }
  
    #last-updated {
  
      margin-top: 10px;
  
      font-size: 12px;
  
      color: #555;
  
    }
  
    .alert {
  
      margin-top: 15px;
  
      padding: 10px;
  
      border-radius: 8px;
  
      background: #ffdddd;
  
      color: #900;
  
      font-weight: bold;
  
    }
  
  </style></head><body>  <div class="container"><h1>🌦️ Dynamic Weather Information System</h1>



<div id="search-box">

  <input type="text" id="city-input" placeholder="Enter city name...">

  <button id="search-btn">Search</button>

  <button id="location-btn">📍 Use My Location</button>

  <button id="refresh-btn">🔄 Refresh Now</button>

</div>



<div id="history"></div>



<div id="weather-result"></div>

<div id="forecast"></div>

  </div>  <script>
  
    const apiKey = "3806dca0dd7b7ebe40d7a1d0cc71eac7"; // 🔑 Replace with your OpenWeather API key
  
    const weatherApiUrl = "https://api.openweathermap.org/data/2.5/weather?units=metric";
  
    const forecastApiUrl = "https://api.openweathermap.org/data/2.5/forecast?units=metric";
  
    const airQualityApiUrl = "https://api.openweathermap.org/data/2.5/air_pollution";
  

  
    const searchBtn = document.getElementById("search-btn");
  
    const cityInput = document.getElementById("city-input");
  
    const locationBtn = document.getElementById("location-btn");
  
    const refreshBtn = document.getElementById("refresh-btn");
  
    const weatherResultDiv = document.getElementById("weather-result");
  
    const forecastDiv = document.getElementById("forecast");
  
    const historyDiv = document.getElementById("history");
  

  
    let searchHistory = JSON.parse(localStorage.getItem("weatherHistory")) || [];
  
    let lastLocation = null;
  

  
    displayHistory();
  

  
    searchBtn.addEventListener("click", () => {
  
      const city = cityInput.value.trim();
  
      if (city) {
  
        fetchWeatherByCity(city);
  
        addToHistory(city);
  
      }
  
    });
  

  
    locationBtn.addEventListener("click", () => {
  
      if (navigator.geolocation) {
  
        navigator.geolocation.getCurrentPosition(
  
          async (position) => {
  
            const lat = position.coords.latitude;
  
            const lon = position.coords.longitude;
  
            try {
  
              const response = await fetch(`${weatherApiUrl}&lat=${lat}&lon=${lon}&appid=${apiKey}`);
  
              const data = await response.json();
  
              fetchAllWeatherData(lat, lon);
  
              addToHistory(data.name);
  
            } catch {
  
              displayError("Failed to detect location.");
  
            }
  
          },
  
          () => displayError("Location access denied.")
  
        );
  
      } else {
  
        displayError("Geolocation not supported.");
  
      }
  
    });
  

  
    refreshBtn.addEventListener("click", () => {
  
      if (lastLocation) {
  
        fetchAllWeatherData(lastLocation.lat, lastLocation.lon);
  
      } else {
  
        displayError("No location to refresh. Search a city first.");
  
      }
  
    });
  

  
    async function fetchWeatherByCity(city) {
  
      try {
  
        const response = await fetch(`${weatherApiUrl}&q=${city}&appid=${apiKey}`);
  
        if (!response.ok) throw new Error("City not found");
  
        const data = await response.json();
  
        const { lat, lon } = data.coord;
  
        fetchAllWeatherData(lat, lon);
  
      } catch {
  
        displayError("City not found. Please try again.");
  
      }
  
    }
  

  
    async function fetchAllWeatherData(lat, lon) {
  
      try {
  
        lastLocation = { lat, lon };
  
        const [weatherData, forecastData, airQualityData] = await Promise.all([
  
          fetch(`${weatherApiUrl}&lat=${lat}&lon=${lon}&appid=${apiKey}`).then(res => res.json()),
  
          fetch(`${forecastApiUrl}&lat=${lat}&lon=${lon}&appid=${apiKey}`).then(res => res.json()),
  
          fetch(`${airQualityApiUrl}?lat=${lat}&lon=${lon}&appid=${apiKey}`).then(res => res.json())
  
        ]);
  

  
        displayCurrentWeather(weatherData, airQualityData);
  
        forecastDiv.innerHTML = ""; // clear before adding
  
        displayForecast(forecastData);
  
        displayTodayTimeline(forecastData); // NEW timeline
  
      } catch {
  
        displayError("Failed to fetch weather data.");
  
      }
  
    }
  

  
    function displayCurrentWeather(weather, aqiData) {
  
      const { name } = weather;
  
      const { country, sunrise, sunset } = weather.sys;
  
      const { icon, description } = weather.weather[0];
  
      const { temp, feels_like, humidity, pressure } = weather.main;
  
      const { speed } = weather.wind;
  
      const windSpeedKmh = (speed * 3.6).toFixed(1);
  
      const visibilityKm = (weather.visibility / 1000).toFixed(1);
  

  
      const aqiValue = aqiData.list[0].main.aqi;
  
      const aqiText = getAqiText(aqiValue);
  

  
      const sunriseTime = new Date(sunrise * 1000).toLocaleTimeString('en-IN', { hour: '2-digit', minute: '2-digit' });
  
      const sunsetTime = new Date(sunset * 1000).toLocaleTimeString('en-IN', { hour: '2-digit', minute: '2-digit' });
  
      const lastUpdatedTime = new Date().toLocaleTimeString('en-IN', { hour: '2-digit', minute: '2-digit' });
  

  
      weatherResultDiv.innerHTML = `
  
        <h2>${name}, ${country}</h2>
  
        <img id="weather-icon" src="https://openweathermap.org/img/wn/${icon}@2x.png" alt="Weather icon">
  
        <p style="font-size: 28px; margin: 5px 0;">${Math.round(temp)}°C (Feels like: ${Math.round(feels_like)}°C)</p>
  
        <p style="text-transform: capitalize;">${description}</p>
  
        <p>Humidity: ${humidity}%</p>
  
        <p>Pressure: ${pressure} hPa</p>
  
        <p>Wind Speed: ${windSpeedKmh} km/h 🌬️</p>
  
        <p>Visibility: ${visibilityKm} km 👁️</p>
  
        <p>🌅 Sunrise: <span style="color:orange;">${sunriseTime}</span></p>
  
        <p>🌇 Sunset: <span style="color:purple;">${sunsetTime}</span></p>
  
        <div class="aqi-info" style="background-color:${aqiText.color};">
  
          Air Quality: ${aqiText.level}
  
        </div>
  
        <p id="last-updated">Last updated at ${lastUpdatedTime}</p>
  
      `;
  

  
      displaySafetyAlerts(weather, aqiData);
  
    }
  

  
    function displayForecast(forecast) {
  
      forecastDiv.innerHTML += "<h3>5-Day Forecast</h3>";
  
      const forecastContainer = document.createElement("div");
  
      forecastContainer.className = "forecast-container";
  

  
      const dailyData = forecast.list.filter((_, index) => index % 8 === 0);
  

  
      dailyData.forEach(day => {
  
        const date = new Date(day.dt * 1000);
  
        const dayString = date.toLocaleDateString("en-IN", { weekday: "short", month: "short", day: "numeric" });
  
        const icon = day.weather[0].icon;
  
        const tempMin = Math.round(day.main.temp_min);
  
        const tempMax = Math.round(day.main.temp_max);
  

  
        const card = document.createElement("div");
  
        card.className = "forecast-card";
  
        card.innerHTML = `
  
          <h4>${dayString}</h4>
  
          <img src="https://openweathermap.org/img/wn/${icon}.png" alt="icon">
  
          <p>Min: ${tempMin}°C</p>
  
          <p>Max: ${tempMax}°C</p>
  
        `;
  
        forecastContainer.appendChild(card);
  
      });
  

  
      forecastDiv.appendChild(forecastContainer);
  
    }
  

  
    function displayTodayTimeline(forecast) {
  
      const todayDiv = document.createElement("div");
  
      todayDiv.innerHTML = "<h3>Today's Timeline</h3>";
  

  
      const timelineContainer = document.createElement("div");
  
      timelineContainer.className = "forecast-container";
  

  
      const todayDate = new Date().toDateString();
  
      let firstRainTime = null;
  

  
      const todayData = forecast.list.filter(item => {
  
        const itemDate = new Date(item.dt * 1000).toDateString();
  
        return itemDate === todayDate;
  
      });
  

  
      todayData.forEach(slot => {
  
        const time = new Date(slot.dt * 1000).toLocaleTimeString("en-IN", {
  
          hour: "2-digit",
  
          minute: "2-digit"
  
        });
  
        const icon = slot.weather[0].icon;
  
        const condition = slot.weather[0].main;
  
        const temp = Math.round(slot.main.temp);
  
        const rain = slot.rain ? slot.rain["3h"] : 0;
  

  
        if (rain > 0 && !firstRainTime) {
  
          firstRainTime = time;
  
        }
  

  
        const card = document.createElement("div");
  
        card.className = "forecast-card";
  
        card.innerHTML = `
  
          <h4>${time}</h4>
  
          <img src="https://openweathermap.org/img/wn/${icon}.png" alt="icon">
  
          <p>${condition} ${rain > 0 ? `🌧️ (${rain} mm)` : ""}</p>
  
          <p>${temp}°C</p>
  
        `;
  
        timelineContainer.appendChild(card);
  
      });
  

  
      todayDiv.appendChild(timelineContainer);
  
      forecastDiv.appendChild(todayDiv);
  

  
      // Add rain alert if found
  
      if (firstRainTime) {
  
        weatherResultDiv.innerHTML += `<div class="alert">☔ Rain expected around ${firstRainTime}. Carry an umbrella!</div>`;
  
      }
  
    }
  

  
    function displaySafetyAlerts(weather, aqiData) {
  
      const { temp } = weather.main;
  
      const aqiValue = aqiData.list[0].main.aqi;
  
      let alerts = "";
  

  
      if (temp > 35) alerts += "<div class='alert'>⚠️ Heat alert: Stay hydrated!</div>";
  
      if (temp < 5) alerts += "<div class='alert'>⚠️ Cold alert: Dress warmly!</div>";
  
      if (aqiValue >= 4) alerts += "<div class='alert'>⚠️ Poor Air Quality. Wear a mask outdoors.</div>";
  

  
      weatherResultDiv.innerHTML += alerts;
  
    }
  

  
    function getAqiText(aqi) {
  
      switch (aqi) {
  
        case 1: return { level: "Good 😀", color: "green" };
  
        case 2: return { level: "Fair 🙂", color: "yellow" };
  
        case 3: return { level: "Moderate 😐", color: "orange" };
  
        case 4: return { level: "Poor 😷", color: "red" };
  
        case 5: return { level: "Very Poor ☠️", color: "purple" };
  
        default: return { level: "Unknown", color: "gray" };
  
      }
  
    }
  

  
    function addToHistory(city) {
  
      if (!searchHistory.includes(city)) {
  
        searchHistory.unshift(city);
  
        if (searchHistory.length > 5) searchHistory.pop();
  
        localStorage.setItem("weatherHistory", JSON.stringify(searchHistory));
  
        displayHistory();
  
      }
  
    }
  

  
    function displayHistory() {
  
      historyDiv.innerHTML = "<strong>Recent Searches:</strong> ";
  
      if (searchHistory.length === 0) return;
  
      searchHistory.forEach(city => {
  
        const btn = document.createElement("button");
  
        btn.textContent = city;
  
        btn.className = "history-btn";
  
        btn.onclick = () => fetchWeatherByCity(city);
  
        historyDiv.appendChild(btn);
  
      });
  

  
      const clearBtn = document.createElement("button");
  
      clearBtn.textContent = "Clear History";
  
      clearBtn.className = "clear-btn";
  
      clearBtn.onclick = () => {
  
        searchHistory = [];
  
        localStorage.removeItem("weatherHistory");
  
        displayHistory();
  
      };
  
      historyDiv.appendChild(clearBtn);
  
    }
  

  
    function displayError(message) {
  
      weatherResultDiv.innerHTML = `<p style="color:red;">${message}</p>`;
  
      forecastDiv.innerHTML = "";
  
    }
  

  
    setInterval(() => {
  
      if (lastLocation) {
  
        fetchAllWeatherData(lastLocation.lat, lastLocation.lon);
  
      }
  
    }, 900000);
  
  </script></body></html>                                                                                                                                                                                 
