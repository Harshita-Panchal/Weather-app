<template>
    <div id="App">
        <main>
            <div class="search-box">
                <input type="search" placeholder="Enter Location" class="search-bar" v-model="state"
                    @keypress.enter="fetch_weather" on-keyup="debounceMethod()" @input="filterState"
                    @focus="modal = true" @keypress="created">
                <div v-if="filtered && modal">
                    <ul class="autocomplete-result" v-on:click="fetch_weather" @focus="input">
                        <li v-for="(filtered, i) in filtered" v-bind:key="i" @click="setState(filtered)">
                            {{ filtered }}
                        </li>
                    </ul>
                </div>
            </div>
            <div class="weather" v-if="weather.location != undefined">
                <div class="location-box">
                    <div class="location"><i class="fa-sharp fa-solid fa-location-dot"></i>
                        {{ weather.location?.name }}, {{ weather.location?.country }}</div>
                    <div class="weather-image"><img id="wicon" alt="weather_icon"
                            v-bind:src="`https:${weather.current?.condition?.icon}`" /></div>
                </div>
                <div class="block">
                    <div>
                        <div class="weather-box">
                            <div class="temp">{{ Math.round(weather.current?.temp_c) }}°c</div>
                            <div class="weather-description">{{ weather.current?.condition?.text }}</div>
                        </div>
                    </div>
                    <div class="more-details">
                        <div class="astro">
                            <p class="sunrise">Sunrise {{ weather.forecast?.forecastday[0]?.astro.sunrise }}</p>
                            <p>Sunset {{ weather.forecast?.forecastday[0]?.astro.sunset }}</p>
                        </div>
                        <div class="current-detail">
                            <p> Chance of rain <span>{{
                                weather.forecast?.forecastday[0]?.day.daily_chance_of_rain
                            }}</span>
                            </p>
                            <p>Humidity <span>{{ weather.current?.humidity }}</span></p>
                        </div>
                        <div class="current-detail">
                            <p>Wind speed <span>{{ weather.current?.wind_kph }}kph</span></p>
                            <p> Pressure <span>{{ weather.current?.pressure_mb }}mb</span></p>
                        </div>
                    </div>
                </div>
                <div class="minmax">
                    <p>max: {{ Math.round(weather.forecast?.forecastday[0]?.day.maxtemp_c) }}°</p>
                    <p>min: {{ Math.round(weather.forecast?.forecastday[0]?.day.mintemp_c) }}°</p>
                </div>
                <hr>
                <div class="currentDay">
                    <p>Today</p>
                    <p class="date">{{ dateBuilder() }} </p>
                </div>
                <div id="hourly-weather">
                    <div class="hourly-weather" v-for="(time, index) in filter" :key="index">
                        <p>{{ getTime(time.time) }}</p>

                        <div class="weather-image-hourly"><img id="wicon-hour" alt="weather_icon"
                                v-bind:src="`https:${weather.current?.condition?.icon}`" /></div>
                        <p>{{ Math.round(weather.forecast?.forecastday[0]?.hour[index]?.temp_c) }} °c</p>
                    </div>
                </div>
                <section id="weekofdays">
                    <div class="Days" v-for="(date, index ) in week " :key="index">
                        <div>
                            <p class="forecast-days"> {{ getday(date.date) }} </p>
                            <p>{{ Math.round(weather.forecast?.forecastday[index]?.day.maxtemp_c) }} °c</p>
                        </div>
                    </div>
                </section>
            </div>
            <div class="title" v-else>Weather App</div>
        </main>
    </div>
</template>

<script>
import moment from 'moment';


export default {
    name: 'App',

    data() {
        return {
            api_key: '1d35dcc8063b4122ad764306233001',
            url_base: 'https://api.weatherapi.com/v1/',
            weather_icon: '//cdn.weatherapi.com/weather/64x64/day/',
            weather: {},

            state: "",
            modal: false,
            states: {},
            filtered: [],
        }
    },
    created() {
        fetch('https://countriesnow.space/api/v0.1/countries/cities', {
            method: "post",
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                "country": "india"
            })
        })
            .then(res => {
                return res.json();
            })
            .then((data) => {
                this.setData(data.data)
                return data
            })
    },

    methods: {
        fetch_weather(e) {
            console.log("test",);
            if (e.key === "Enter" || e.type === "click") {
                fetch(`${this.url_base}forecast.json?key=${this.api_key}&q=${this.state}&days=7&aqi=yes&alerts=no`)
                    .then(res => {
                        return res.json();
                    }).then(this.setResults);
            }
        },
        setResults(results) {
            this.weather = results
            console.log(results, "setresults")
            console.log(this.weather, "weather data");
        },
        dateBuilder() {
            let d = new Date();
            let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "Octomber", "November",
                "December",
            ]
            // let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]
            // let day = days[d.getDay()];
            let month = months[d.getMonth()];
            let date = d.getDate();
            let year = d.getFullYear();
            return ` ${date} ${month} ${year}`;
        },
        getTime(datetime) {
            return moment(datetime).format('HH:mm')
        },
        getday(date) {
            console.log(date, ' hi')
            return moment(date).format('dddd')
        },
        getApiData() {
            console.log('fetch')
        },

        debounce(func, delay) {
            let timer;
            return function () {
                let context = this
                let arg = arguments;
                clearTimeout(timer);
                timer = setTimeout(() => {
                    func.apply(context, arg);
                }, delay)
            }
        },

        debounceMethod() {
            this.debounce(this.getApiData, 300)
        },

        setData(data) {
            this.states = data

        },

        filterState() {
            this.filtered = this.states.filter(state => {
                return state.toLowerCase().startsWith(this.state.toLowerCase());
            })
        },
        setState(state) {
            this.state = state;
            this.modal = false
        },

    },

    computed: {
        filter() {
            return this.weather?.forecast?.forecastday[0]?.hour;
        },
        week() {
            return this.weather?.forecast?.forecastday;
        },
        moment: function (time) {
            return moment(time).format("hh:mm")
        },


    }
}

</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: sans-serif;
    color: white;
}

main {
    min-height: 100vh;
    max-width: 1024px;
    margin: 0 auto;
    padding: 25px;
    background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
}

.search-box {
    width: 100%;
    margin-bottom: 30px;
}

.search-box .search-bar {
    display: block;
    width: 100%;
    padding: 15px;
    color: #313131;
    font-size: 20px;
    box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
    border: none;
    outline: none;
    background-color: rgba(255, 255, 255, 0.5);
    border-radius: 10px 0;
    transition: 0.4s;
}

.location-box .location {
    color: white;
    font-size: 32px;
    font-weight: 500;
    text-align: center;
}

.weather-box .date {
    color: white;
    font-size: 20px;
    font-weight: 300;
    font-style: italic;
    text-align: center;
}

.weather-box {
    text-align: center;
}

.weather-box .temp {
    display: inline-block;
    color: white;
    font-size: 102px;
    font-weight: 900;

    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
    border-radius: 16px;
    margin: 30px 0;
}

.weather-box .weather-description {
    color: white;
    font-size: 16px;
    font-weight: 600;
    font-style: italic;
}

.currentDay {
    display: flex;
    justify-content: space-between;
}

.minmax {
    display: flex;
    padding: 5px 0;
}

.minmax p {
    padding: 0 10px;
    font-size: 12px;
}

.weather-image {
    display: flex;
    justify-content: center;
}

.weather-image img {
    width: 150px;
}

.sunrise {
    padding-right: 45px;
}

#hourly-weather {
    display: flex;
    justify-content: space-between;
    overflow-x: scroll;
    scrollbar-width: none;
    margin: 30px 0px;
    background-color: #5a5553;
}

#hourly-weather::-webkit-scrollbar {
    display: none;
}

.hourly-weather {
    margin-right: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.hourly-weather p {
    padding: 6px 0;
    width: 75px;
    text-align: center;
}

.block {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: space-evenly;
}

@media (max-width: 600px) {
    .search-box .search-bar {
        padding: 10px 15px;
    }

    .weather-box .temp {
        font-size: 64px;
        margin: 20px 0;
    }

    .location-box .location {
        font-size: 20px;
    }

    .location-box .date {
        font-size: 16px;
    }

    #weekofdays {
        display: block;
        text-align: end;
    }

}

hr {
    margin: 10px 0;
}

.daily-weather {
    display: flex;
    justify-content: space-between;
}

.astro {
    display: flex;
    justify-content: space-between;
}

.astro p {
    font-size: 13px;
}

.current-detail {
    display: flex;
    justify-content: space-between;
}

.current-detail p {
    font-size: 15px;
    padding: 6px 0;
}

.more-details {
    padding: 15px;
}

.more-details p span {
    display: flex;
    flex-direction: column;
}

#wicon-hour {
    width: 45px;
}

.title {
    font-size: 45px;
    font-style: italic;
    text-align: center;
}

li {
    background: gainsboro;
    color: black;
    margin: 5px 0;
    list-style: none;
    cursor: pointer;
}

#weekofdays {
    display: flex;
    justify-content: space-between;
    text-align: center;
    background-color: #5a5553;
    padding: 10px;
}

.forecast-days {
    padding: 10px;
}
</style>
