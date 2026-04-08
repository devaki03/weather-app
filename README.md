<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Weather App</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #87CEEB, #4682B4);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .weather-card {
            background: white;
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            max-width: 450px;
            width: 100%;
            text-align: center;
        }

        .search-box {
            margin-bottom: 30px;
        }

        .search-input {
            width: 70%;
            padding: 15px;
            border: 2px solid #ddd;
            border-radius: 30px;
            font-size: 16px;
            outline: none;
            margin-right: 10px;
        }

        .search-btn, .location-btn {
            padding: 15px 25px;
            border: none;
            border-radius: 30px;
            background: #4CAF50;
            color: white;
            font-size: 16px;
            cursor: pointer;
            transition: background 0.3s;
        }

        .search-btn:hover, .location-btn:hover {
            background: #45a049;
        }

        .loading {
            font-size: 18px;
            color: #666;
            display: none;
        }

        .error {
            color: #e74c3c;
            font-size: 16px;
            display: none;
        }

        .weather-icon {
            font-size: 80px;
            margin: 20px 0;
            color: #3498db;
        }

        .city {
            font-size: 28px;
            font-weight: bold;
            color: #2c3e50;
            margin-bottom: 10px;
        }

        .temperature {
            font-size: 64px;
            font-weight: 300;
            color: #e74c3c;
            margin-bottom: 10px;
        }

        .description {
            font-size: 20px;
            color: #7f8c8d;
            margin-bottom: 30px;
            text-transform: capitalize;
        }

        .details {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            margin-top: 20px;
        }

        .detail {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
        }

        .detail i {
            font-size: 24px;
            color: #3498db;
            margin-bottom: 5px;
        }

        .detail-value {
            font-size: 20px;
            font-weight: bold;
            color: #2c3e50;
        }

        @media (max-width: 480px) {
            .search-input {
                width: 60%;
                margin-bottom: 10px;
            }
            
            .temperature {
                font-size: 48px;
            }
        }
    </style>
</head>
<body>
    <div class="weather-card">
        <div class="search-box">
            <input type="text" id="cityInput" class="search-input" placeholder="Enter city name">
            <button class="search-btn" onclick="getWeather()">
                <i class="fas fa-search"></i> Search
            </button>
            <button class="location-btn" onclick="getLocation()">
                <i class="fas fa-map-marker-alt"></i> My Location
            </button>
        </div>

        <div id="loading" class="loading">
            <i class="fas fa-spinner fa-spin"></i> Loading...
        </div>
        <div id="error" class="error"></div>

        <div id="weatherInfo" style="display: none;">
            <div class="weather-icon" id="weatherIcon"></div>
            <div class="city" id="city"></div>
            <div class="temperature" id="temp"></div>
            <div class="description" id="description"></div>
            
            <div class="details">
                <div class="detail">
                    <i class="fas fa-thermometer-half"></i>
                    <div class="detail-value" id="feelsLike">--</div>
                    <div>Feels Like</div>
                </div>
                <div class="detail">
                    <i class="fas fa-tint"></i>
                    <div class="detail-value" id="humidity">--</div>
                    <div>Humidity</div>
                </div>
                <div class="detail">
                    <i class="fas fa-wind"></i>
                    <div class="detail-value" id="wind">--</div>
                    <div>Wind</div>
                </div>
                <div class="detail">
                    <i class="fas fa-eye"></i>
                    <div class="detail-value" id="pressure">--</div>
                    <div>Pressure</div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 👈 CHANGE THIS LINE WITH YOUR API KEY
        const API_KEY = 'fa95ca2f36db46b190848e05a97158f6'; 
        // Get free key: https://openweathermap.org/api

        const API_URL = 'https://api.openweathermap.org/data/2.5/weather';

        async function getWeather() {
            const city = document.getElementById('cityInput').value.trim();
            if (!city) {
                showError('Please enter a city');
                return;
            }
            await fetchWeather(city);
        }

        async function getLocation() {
            showLoading();
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(async (position) => {
                    const lat = position.coords.latitude;
                    const lon = position.coords.longitude;
                    const res = await fetch(`${API_URL}?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`);
                    const data = await res.json();
                    displayWeather(data);
                }, () => {
                    showError('Cannot get location');
                });
            }
        }

        async function fetchWeather(city) {
            showLoading();
            try {
                const res = await fetch(`${API_URL}?q=${city}&appid=${API_KEY}&units=metric`);
                if (!res.ok) throw new Error('City not found');
                const data = await res.json();
                displayWeather(data);
            } catch (err) {
                showError(err.message);
            }
        }

        function displayWeather(data) {
            document.getElementById('city').textContent = `${data.name}, ${data.sys.country}`;
            document.getElementById('temp').textContent = `${Math.round(data.main.temp)}°C`;
            document.getElementById('description').textContent = data.weather[0].description;
            document.getElementById('feelsLike').textContent = `${Math.round(data.main.feels_like)}°C`;
            document.getElementById('humidity').textContent = `${data.main.humidity}%`;
            document.getElementById('wind').textContent = `${data.wind.speed} m/s`;
            document.getElementById('pressure').textContent = `${data.main.pressure} hPa`;

            // Weather icon
            const icon = data.weather[0].icon;
            document.getElementById('weatherIcon').innerHTML = getWeatherIcon(icon);

            hideLoading();
            hideError();
            document.getElementById('weatherInfo').style.display = 'block';
        }

        function getWeatherIcon(icon) {
            const icons = {
                '01d': 'fas fa-sun', '01n': 'fas fa-moon',
                '02d': 'fas fa-cloud-sun', '02n': 'fas fa-cloud-moon',
                '03d': 'fas fa-cloud', '03n': 'fas fa-cloud',
                '04d': 'fas fa-clouds', '04n': 'fas fa-clouds',
                '09d': 'fas fa-cloud-rain', '09n': 'fas fa-cloud-rain',
                '10d': 'fas fa-cloud-sun-rain', '10n': 'fas fa-cloud-moon-rain',
                '11d': 'fas fa-bolt', '11n': 'fas fa-bolt',
                '13d': 'fas fa-snowflake', '13n': 'fas fa-snowflake',
                '50d': 'fas fa-smog', '50n': 'fas fa-smog'
            };
            return `<i class="${icons[icon] || 'fas fa-cloud'}"></i>`;
        }

        function showLoading() {
            document.getElementById('loading').style.display = 'block';
            document.getElementById('weatherInfo').style.display = 'none';
            hideError();
        }

        function hideLoading() {
            document.getElementById('loading').style.display = 'none';
        }

        function showError(msg) {
            hideLoading();
            document.getElementById('error').textContent = msg;
            document.getElementById('error').style.display = 'block';
        }

        function hideError() {
            document.getElementById('error').style.display = 'none';
        }

        // Enter key to search
        document.getElementById('cityInput').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') getWeather();
        });
    </script>
</body>
</html>
