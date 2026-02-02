<!--HTML-->
<template>
  <!--Dynamic class for checking whether warm/cold for background-->
  <div id = "app" :class="typeof weather.main != 'undefined' && weather.main.temp >16? 'warm':''">
    <main>
      <!--Search-->
      <div class = "searchBox">
        <input type="text"
          class="searchBar"
          placeholder="Enter your city/town here..."
          v-model="query"
          @keypress="fetchWeather"
          />
          {{ query }}
      </div>

      <!--Weather display-->
      <!--Let API send data before app shows temparature-->
      <div class="weatherWrap" v-if="typeof weather.main != 'undefined'">
        <div class= 'locationBox' >
          <div class='location'>{{  weather.name }},{{ weather.sys.country }}</div>
          <div class='date'>{{ dateBuilder() }}</div>
        </div>

        <div class='weatherBox'>
          <div class='temp'>
            {{ Math.round(weather.main.temp)}}°C
          </div>
          <div class='weather'>
            {{ weather.weather[0].main }}
          </div>
        </div>
      </div>
      
      
      <div class="forecastTableWrap" v-if="forecast.length > 0">
        <table class="forecastTable">
          <thead>
            <tr>
              <th>Day</th>
              <th>Temp</th>
              <th>Weather</th>
            </tr>
          </thead>
          <tbody>
            <!--Create row for every day-->
            <tr v-for="day in forecast" :key="day.dt">
              <td>{{ new Date(day.dt * 1000).toLocaleDateString('en-ZA', {weekday: 'short'}) }}</td>
              <td>{{ Math.round(day.main.temp) }}°C</td>
              <td>{{ day.weather[0].main }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!--Error-->
      <div class= 'errorWrap' v-else-if= "weather.cod == 404">
          <div class="errorMessage">
            <p class = "errorStyle1">{{ query }} not found :C</p>
            <p class = "errorStyle">Try searching for your closest city/town.</p>
          </div>
      </div>

    <!--Raindrops-->
    <div v-if="weather.weather && weather.weather[0].main === 'Rain'" class="rainContainer">
      <div 
        v-for="n in 50" 
        :key="n" 
        class="rainDrop"
        :style="{ 
          left: Math.random() * 100 + '%', 
          animationDuration: (Math.random() * 1 + 0.5) + 's',
          animationDelay: Math.random() * 2 + 's' 
        }">
      </div>
    </div>
  </main>
  </div>
</template>

<!--SCRIPT-->
<script>
export default {
  name: 'App',
  data(){
    return{
      api_key: 'e0ea77b0476f1838f61a4ae2f2812fab',
      url_base:'https://api.openweathermap.org/data/2.5/',
      query:'',//user input
      weather:{},//store the current weather results
      forecast:[]//store the week forecast results(5days limit API)
    }
  },

  methods:{
    //fetch current weather
    fetchWeather(e){
      if(e.key == "Enter"){
        //Remove old weather&focast for new one
        this.weather = {};
        this.focast = [],
        fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
        .then(res=> res.json())//Convert raw data to JSON
        .then(this.setResults);

    //5 days forecast
    fetch(`${this.url_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`)
          .then(res => res.json())
          .then(results => {
            // We check if results.list exists before filtering to avoid errors
            if (results.list) {
              this.forecast = results.list.filter(item => item.dt_txt.includes("12:00:00"));
            }else{
              this.forecast=[];
            }
          });
        }
  },
  //method to update weather object with search results
  setResults(results){
    this.weather = results;
  },

  //Date creation
  dateBuilder(){
    let d = new Date();
    let months = [" January "," February ", " March ", " April ", " May ", " June ", " July ", " August ", " September ", " October ", " November ", " December "];
    let days = [" Sunday ", " Monday ", " Tuesday ", " Wednesday ", " Thursday ", " Friday ", " Saturday "];

    let day = days[d.getDay()];
    let date = d.getDate();
    let month = months[d.getMonth()];
    let year = d.getFullYear();

    return `${day}${date}${month}${year}`;
  }
},
}
</script>

<!--STYLE-->
<style>
/*Roboto fonts import */
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,400&display=swap');
*{
  margin:0;
  padding:0;
  box-sizing: border-box;
  color:white;
}
.forecastTableWrap {
  margin-top: 50px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0px 0px 16px rgba(0,0,0,0.25);
}

.forecastTable {
  width: 100%;
  border-collapse: collapse;
  color: #FFF;
}

.forecastTable th, .forecastTable td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid rgb(207, 202, 202);
}

.forecastTable th {
  text-transform: uppercase;
  font-size: 14px;
}
.forecastTable td{
  opacity:0.8;
}
.rainContainer {
  position: absolute;
  top:0px;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: 1;
  pointer-events: none;
}

.rainDrop {
  position: absolute;
  background-color: rgba(255, 255, 255, 0.4);
  width: 2px;
  height: 15px;
  top: -20px;
  animation: fall linear infinite;
}

.rainDrop:nth-child(odd) { background-color: rgba(255, 255, 255, 0.2); }

@keyframes fall {
  to {
    transform: translateY(100vh);
  }
}

.errorStyle1{
  color:red;
}

body{
  font-family: 'Roboto','sans-serif';
}

#app{
    background-image: url('./assets/cold.png');
    background-size: cover;
    background-position: bottom;
    transition:0.4s;
    background-color: lightblue;
}

#app.warm{
    background-image: url('./assets/warm.png');
    background-color: orange;
}

main{
  min-height: 100vh;
  padding :25px;
  background-image: linear-gradient(to bottom,rgba(0,0,0,0.25),rgba(0,0,0,0.75));
}

.searchBox{
  width: 100%;
  margin-bottom: 30px;
  
}

.searchBox .searchBar{
  display:block;
  width:100%; 
  padding:15px;

  color:#313131;
  font-size:20px;

  appearance: none;
  border:none;
  outline:none;
  background:none;

  box-shadow: 0px 0px 8px rgba(0,0,0,0.25);
  background-color: rgba(255,255,255,0.15);
  border-radius:50px;
  transition: 0.4s;
}

.searchBox .searchBar:focus{
  box-shadow:0px 0px 16px;
  background-color: rgba(255,255,255,0.75);
  border-radius:50px;
}

.locationBox .location{
  color:#FFF;
  font-size: 32px;
  font-weight:500;
  text-align: center;
  text-shadow: 1px 3px rgba(0,0,0,0.25);
}

.locationBox .date{
  color:#FFF;
  font-size: 20px;
  font-weight:300;
  font-style:italic;
  text-align: center;
}

.weatherBox{
  text-align:center;
}

.weatherBox .temp{
  display:inline-block;
  padding: 10px 25px;
  color:#FFF;
  font-weight:900;

  text-shadow: 3px 6px rgba(0,0,0,0.25);
  background-color: rgba(255,255,255,0.25);
  border-radius:16px;
  margin: 30px 0px;

  box-shadow: 3px 6px rgba (0,0,0,0.25);
}

.weatherBox .weather{
  color:#FFF;
  font-size:48px;
  font-weight:700;
  font-style:italic;
  text-shadow:3px 6px rgba(0,0,0,0.25);
}
</style>
