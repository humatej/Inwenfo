<template>
    <div class="container">
        <div id="search-bar" class="row">
            <div class="col-md-7 mx-auto">
                <div class="input-group">
                    <input type="text" class="form-control" v-model="searchText" placeholder="Search for...">
                </div>
            </div>
        </div>
        <div id="search-result" class="row">
            <div class="col-md-7 mx-auto">
                <!-- temp search result list -->                
                <table v-show="!showFullResult && showPlaceResult" class="table table-hover table-dark table-striped">
                    <thead class="bg-primary">
                        <tr>
                            <th class="col-6" scope="col">City</th>
                            <th class="col-6" scope="col">Country</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="result in searchPlaceResult" :key="result"
                            class="my-auto">
                            <th class="align-middle" scope="row">
                                <a @click="showInfo(result.latitude, result.longitude)">
                                    {{ result.name }}
                                </a>
                            </th>
                            <td class="align-middle">
                                <img :src='"https://hatscripts.github.io/circle-flags/flags/" + result.country_code.toLowerCase() + ".svg"' width="48">
                                {{ result.country }}
                            </td>
                        </tr>
                    </tbody>
                </table>
                <!-- weather data of the region -->
                <div v-show="showFullResult" class="bg-dark">
                    <h1 class="ps-2 pt-2 text-light">Temperature in the region</h1>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        :options="searchWeatherResult.options"
                        :series="searchWeatherResult.series"
                    >
                    </apexchart>
                </div>
            </div>
        </div>
    </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue'
import { ref } from 'vue'


import VueApexCharts from "vue3-apexcharts";

export default defineComponent({
    data(){
        const searchText = ref<string>('')
        const searchPlaceResult = ref<object>({})
        const searchWeatherResult = ref<any>({
            options:{
                chart:{
                    id:"weather-chart",
                    type:"area",
                    foreColor: "#ccc",
                },
                markers: {
                    size: 5,
                    colors: ["#000524"],
                    strokeColor: "#00BAEC",
                    strokeWidth: 3,
                },
                xaxis:{
                    type: 'datetime',
                    tickAmount: 7,
                },
                tooltip: {
                    x: {
                        format: 'HH:mm dd/MM/yyyy'
                    },
                },
                dataLabels: {
                    enabled: false,
                },
                yaxis:{
                    name:"Temperature",
                    tickAmount: 3
                },
                stroke: {
                    curve: 'smooth',
                    color: "#fff",
                },
                fill: {
                    type: 'gradient',
                    gradient:{
                        opacityFrom: 0.7,
                        opacityTo: 0.0,
                    }
                },
            },
            series:[
                {
                    name:"temp",
                    data:[],
                }
            ],
            
        })
        const showFullResult = ref<boolean>(false)
        const showPlaceResult = ref<boolean>(false)

        return{
            searchText,
            searchPlaceResult,
            searchWeatherResult,
            showFullResult,
            showPlaceResult,
        }
    },
    components: {
        apexchart: VueApexCharts,
    },
    watch: {
        async searchText(newVal){
            console.log(newVal);
            if(newVal != ''){
                const response = await fetch("https://geocoding-api.open-meteo.com/v1/search?name=" + newVal)
                const data = await response.json()
                if(data.results){
                    this.showPlaceResult = true
                    this.searchPlaceResult = data.results
                    if(data.results.length > 5){
                        this.searchPlaceResult = data.results.slice(0, 5)
                    }
                }
                else{
                    this.showPlaceResult = false
                }
            }
            else if(newVal == ''){
                this.showFullResult = false
                this.showPlaceResult = false
                this.searchWeatherResult.options.xaxis.categories = []
                this.searchWeatherResult.series[0].data = []
                this.searchPlaceResult = {}
            }
        },
    },
    methods:{
        async showInfo(longitude:number, latitude:number){
            this.showFullResult = true
            const response = await fetch("https://api.open-meteo.com/v1/forecast?latitude=" + latitude + "&longitude=" + longitude + "&hourly=temperature_2m,relativehumidity_2m,windspeed_10m")
            const data = await response.json()
            //and time an temperature to array of object pairs
            data.hourly.time.forEach((time:number, index:number) => {
                console.log(data.hourly.temperature_2m[index])
                this.searchWeatherResult.series[0].data = [...this.searchWeatherResult.series[0].data,
                                                            {  x:time, 
                                                               y:data.hourly.temperature_2m[index]
                                                            }
                                                          ]
            })
        }
    }

})
</script>
<style lang="scss">
    #search-result{
        overflow: hidden;
        position: relative;
        z-index: 1;
    }
</style>