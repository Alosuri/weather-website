<script>
export default {
  data() {
    return {
      cities: [],
      message: "",
	  weatherData: {
		city: "",
		country: "",
		temperature: "",
		humidity: "",
		precipitation: "",
		unites: "°C",
		icon: "wi wi-0",
		hourlyWeather: {
			
		}
	  }
    }
  },

  methods: {
    	searchCity() {
    	  fetch('https://geocoding-api.open-meteo.com/v1/search?name=' + this.message +'&count=20&language=en&format=json')
		  		.then(res => res.json())
		  		.then((out) => {
					this.cities = []
		  			this.writeCity(out.results, this.cities)
		  	}).catch(err => console.log(err));
    	},

    	writeCity(value, counties) {
		  	value.forEach(function (data, i) {
				counties[i] = data
		  	})
		},
		
		clickCity(data) {
			this.cities = []
			this.message = ""
			this.weatherData.city = data.name
			
			fetch("https://api.open-meteo.com/v1/forecast?latitude=" + data.latitude+ "&longitude=" + data.longitude + "&current=temperature_2m,relative_humidity_2m,is_day,precipitation,weather_code,wind_speed_10m&hourly=temperature_2m,weather_code,is_day&timeformat=unixtime&forecast_hours=24")
				.then(res => res.json())
				.then(out => {
					this.weatherData.temperature = out.current.temperature_2m
					this.weatherData.unites = out.current_units.temperature_2m
					this.weatherData.icon = out.current.is_day ? "wi " + "wi-" + out.current.weather_code : "wi " + "wi-night-" + out.current.weather_code
					this.weatherData.country = data.country	
					this.weatherData.humidity = out.current.relative_humidity_2m
					this.weatherData.precipitation = out.current.precipitation
					this.weatherData.wind = out.current.wind_speed_10m

					for(let x in out.hourly.time) {
						let date = String(new Date(out.hourly.time[x] * 1000).getHours() + ":" + ("0" + new Date(out.hourly.time[x] * 1000).getMinutes()))
						this.weatherData.hourlyWeather[x] = [date, out.hourly.temperature_2m[x] + "" + out.current_units.temperature_2m, (out.hourly.is_day[x] ? "wi " + "wi-" + out.hourly.weather_code[x] : "wi " + "wi-night-" + out.hourly.weather_code[x])];
					}

					console.log(this.weatherData.hourlyWeather)
			})
		} 
	},

}

</script>

<template>
  <main>
		<div class="information">
			<div class="searchCity">
				<input placeholder="Search for your city.." type="search" v-model="message" @input="searchCity(message)"/>

				<div class="citiesList" v-if="this.cities != '' && this.message.length > 2"> 
					<ul>
						<template v-for="item in this.cities">
							<li @click="clickCity(item)" style="display: flex; flex-direction: row; justify-content:space-between;">
								<div style="display: flex; gap: 20px">
									<a>{{ item.name }}</a>
									<a>{{ item.admin1 }}</a>
								</div>

								<div style="display: flex; align-items: center;">
									<a>{{ item.country_code }}</a>
									<img v-bind:src="item.country_code != undefined ? 'https://flagsapi.com/' + item.country_code + '/shiny/16.png' : ''">
								</div>
							</li>
						</template>
					</ul>
				</div>
			</div>

			<div style="display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%;"  v-if="this.weatherData.city == '' && this.message.length <= 2">
				<img src="./assets/logo.png" alt="Logo" style="width: 200px;">
				<h1 style="font-weight: 600; font-size: 40px;">Weather</h1>
				<a href="https://github.com/alosuri" style="display: flex; flex-direction: row; justify-content: center; gap: 10px; color: #aaa; text-decoration: none;">
					<img src="./assets/github-mark-white.png" alt="Github" style="width: 20px; height: 20px;">
					<p>Created by alosuri</p>
				</a>
			</div>

			<div class="generalInfo" v-if="this.weatherData.city != '' && this.message.length <= 2">
				<div style="text-align: left;">
					<h1 v-text="this.weatherData.city" style="font-weight: 600;"/>
					<h1 v-text="this.weatherData.country" style="font-weight: 300;"/>
				</div>

				<div style="display: flex; gap: 20px">
					<a class="temperature" style="font-size: 40px; font-weight: 600;">{{ this.weatherData.temperature + this.weatherData.unites }}</a>
					<i v-bind:class="this.weatherData.icon" style="font-size: 35px;"></i>
				</div>

			</div>

			<div style="display: flex; flex-direction: row; gap: 20px" v-if="this.weatherData.city != '' && this.message.length <= 2">
				<p><b style="font-weight: 600;">Precipitation:</b> {{ this.weatherData.precipitation }}%</p>
				<p><b style="font-weight: 600;">Humidity:</b> {{ this.weatherData.humidity }}%</p>
				<p><b style="font-weight: 600;">Wind:</b> {{ this.weatherData.wind }} km/h</p>
			</div>

			<h1 style="font-size: 20px; font-weight: 600;" v-if="this.weatherData.city != '' && this.message.length <= 2">Hourly weather:</h1>

			<div class="hourlyWeather" v-if="this.weatherData.city != '' && this.message.length <= 2">
				<ul>
					<template v-for="item in this.weatherData.hourlyWeather">
						<li>
							<a class="time">{{ item[0] }}</a>
							<i v-bind:class="item[2]" style="font-size: 35px;"></i>
							<a class="temperature">{{ item[1] }}</a>
						
						</li>
					</template>
				</ul>
			</div>
		</div>

	</main>
</template>

<style scoped>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		width: 100vw;
		height: 100vh;
		background-image: url("./assets/bg.jpg");
		background-size: cover;
		overflow-y: hidden;
		overflow-x: hidden;
	}

	.information {
		display: flex;
		flex-direction: column;
		width: 40rem;
		height: 35rem;
		bottom: 0;
		justify-content: space-between;
		align-items: center;
		background-color: rgba(34, 34, 34);
		border-radius: 10px;
		padding: 50px;
	}

	.searchCity {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		width: 100%;
		z-index: 1;
	}

	.searchCity input {
		border-radius: 5px;
		padding: 10px;
		outline-style: none;
		color: #fff;
		border: none;
		width: 30rem;	
		text-align: center;
		background-color: #303030;

	}

	.searchCity input::placeholder {
		color: #aaa;
	}

	.citiesList {
		display: flex;
		flex-direction: row;
		width: 30rem;
		align-items: center;
		justify-content: center;
		padding-top: 20px;
		z-index: 0;
	}

	.citiesList ul{
		border-radius: 10px;
		padding: 10px;
		outline-style: none;
		color: #aaa;
		border: none;
		width: 30rem;
		background-color: #303030;
		padding: 15px;
		max-height: 20rem;
		list-style: none;
		overflow: auto;
	}

	.citiesList li {
		display: flex;
		flex-direction: row;
		align-items: center;
		cursor: pointer;
	}

	.citiesList li a {
		font-weight: 500;
	}
	.generalInfo {
		width: 30rem;
		display: flex;
		flex-direction: row;
		align-items: end;
		font-size: 10px;
		line-height: 25px;
		text-align: center;
		justify-content: space-between;
	}

	.hourlyWeather {
		display: flex;
		width: 100%;
		border-radius: 10px;
		background: #303030;
		padding: 20px;
	}
  
	 .hourlyWeather ul {
		display: flex; 
		flex-direction: row;
		list-style: none; 
		overflow-x: auto; 
		overflow-y: hidden;
		padding: 0;
		padding-bottom: 10px;
	 } 
  
	 .hourlyWeather li { 
		display: flex; 
		flex-direction: column; 
		justify-content: center;
		align-items: center;
		min-width: 100px;
		gap: 5px;
	 } 
  
	 .hourlyWeather .temperature { 
		 font-weight: 600; 
		 font-size: 20px; 
	 } 
  
	 .hourlyWeather .time { 
		 font-size: 20px; 
	 }

	 @media (max-width: 720px) {
		.information {
			width: 85%;
			padding: 30px;
		}

		.searchCity input {
			width: 100%;
		}

		.citiesList {
			width: 100%;
		}

		.generalInfo {
			flex-direction: column;
			justify-content: center;
			align-items: center;
			gap: 20px;
		}
	}
</style>