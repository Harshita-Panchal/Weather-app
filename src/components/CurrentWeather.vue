<template>
<header>
    <div class="weather">
                <div class="location-box" v-if="weather.location != undefined">
                    <div class="location"> {{ weather.location?.name }}, {{ weather.location?.country }}</div>
                    <div class="weather-image"><img id="wicon" alt="weather_icon"
                            v-bind:src="`https:${weather.current?.condition?.icon}`" /></div>
                </div>
                <div class="weather-box">
                    <div class="temp">{{ Math.round(weather.current?.temp_c) }}°c</div>
                    <div class="date">{{ dateBuilder() }} </div>
                    <div class="weather-description">{{ weather.current?.condition?.text }}</div>
                </div>
                <div class="minmax">
                    <p>max: {{ Math.round(weather.forecast?.forecastday[0]?.day.maxtemp_c) }}°</p>
                    <p>min: {{ Math.round(weather.forecast?.forecastday[0]?.day.mintemp_c) }}°</p>
                </div>
    </div>
</header>
</template>

<script>
export default{
    name: "currentWeather",
}
</script>

<style scoped>

</style>