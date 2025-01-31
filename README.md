<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather Report</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }

        body {
            background: #f0f2f5;
            padding: 20px;
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
        }

        .search-box {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            display: flex;
            gap: 10px;
        }

        .search-input {
            flex: 1;
            padding: 10px;
            font-size: 16px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }

        .search-button {
            padding: 10px 20px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        .weather-card {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            display: none;
        }

        .current-weather {
            display: flex;
            align-items: center;
            margin-bottom: 20px;
        }

        .weather-icon {
            width: 100px;
            margin-right: 20px;
        }

        .forecast {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
            gap: 10px;
        }

        .forecast-item {
            background: #f8f9fa;
            padding: 10px;
            border-radius: 5px;
            text-align: center;
        }

        .error {
            color: red;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="search-box">
            <input type="text" class="search-input" placeholder="Enter city name">
            <button class="search-button" onclick="searchWeather()">Search</button>
        </div>
        <div class="error" id="error-message"></div>

        <div class="weather-card" id="current-weather">
            <div class="current-weather">
                <img class="weather-icon" id="weather-icon" src="" alt="Weather Icon">
                <div>
                    <h1 id="city-name"></h1>
                    <p id="temperature"></p>
                    <p id="weather-description"></p>
                    <p>Humidity: <span id="humidity"></span>%</p>
                    <p>Wind Speed: <span id="wind-speed"></span> m/s</p>
                </div>
            </div>
            
            <h2>5-Day Forecast</h2>
            <div class="forecast" id="forecast"></div>
        </div>
    </div>

    <script>
        const API_KEY = '45295d63d1fe48834be91b4ffbc6f2c7'; // Your OpenWeatherMap API Key

        async function getWeatherData(city) {
            try {
                // Fetch current weather
                const currentResponse = await fetch(
                    `https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=${API_KEY}`
                );
                if (!currentResponse.ok) throw new Error('City not found');

                const currentData = await currentResponse.json();

                // Fetch 5-day forecast
                const forecastResponse = await fetch(
                    `https://api.openweathermap.org/data/2.5/forecast?q=${city}&units=metric&appid=${API_KEY}`
                );
                if (!forecastResponse.ok) throw new Error('Forecast data not available');

                const forecastData = await forecastResponse.json();

                return { current: currentData, forecast: forecastData };
            } catch (error) {
                console.error("Weather Fetch Error:", error);
                throw error;
            }
        }

        function displayWeather(data) {
            document.getElementById('current-weather').style.display = 'block';
            document.getElementById('error-message').textContent = '';

            document.getElementById('city-name').textContent = `${data.current.name}, ${data.current.sys.country}`;
            document.getElementById('temperature').textContent = `${Math.round(data.current.main.temp)}°C`;
            document.getElementById('weather-description').textContent = data.current.weather[0].main;
            document.getElementById('humidity').textContent = data.current.main.humidity;
            document.getElementById('wind-speed').textContent = data.current.wind.speed;

            document.getElementById('weather-icon').src = 
                `https://openweathermap.org/img/wn/${data.current.weather[0].icon}@2x.png`;

            const forecastElement = document.getElementById('forecast');
            forecastElement.innerHTML = '';

            const dailyForecasts = {};

            // Group forecast by date and pick the closest time to 12 PM
            data.forecast.list.forEach(item => {
                const date = new Date(item.dt * 1000).toDateString();
                const hour = new Date(item.dt * 1000).getHours();

                if (!dailyForecasts[date] || Math.abs(hour - 12) < Math.abs(dailyForecasts[date].hour - 12)) {
                    dailyForecasts[date] = { ...item, hour };
                }
            });

            // Display 5-day forecast
            Object.values(dailyForecasts).slice(0, 5).forEach(forecast => {
                const date = new Date(forecast.dt * 1000);
                
                const forecastItem = document.createElement('div');
                forecastItem.className = 'forecast-item';
                forecastItem.innerHTML = `
                    <p>${date.toLocaleDateString('en-US', { weekday: 'short' })}</p>
                    <img src="https://openweathermap.org/img/wn/${forecast.weather[0].icon}.png" alt="Weather Icon">
                    <p>${Math.round(forecast.main.temp)}°C</p>
                `;
                
                forecastElement.appendChild(forecastItem);
            });
        }

        function searchWeather() {
            const city = document.querySelector('.search-input').value;
            const errorElement = document.getElementById('error-message');

            if (!city) {
                errorElement.textContent = 'Please enter a city name';
                return;
            }

            getWeatherData(city)
                .then(data => displayWeather(data))
                .catch(error => {
                    errorElement.textContent = error.message;
                    document.getElementById('current-weather').style.display = 'none';
                });
        }
    </script>
</body>
</html>
