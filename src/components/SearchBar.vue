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
                <table v-show="!showFullResult && showPlaceResult" class="my-bg table table-hover table-striped">
                    <thead class="text-light">
                        <tr>
                            <th class="col-6" scope="col">City</th>
                            <th class="col-6" scope="col">Country</th>
                        </tr>
                    </thead>
                    <tbody class="rounded-bottom">
                        <tr v-for="result in searchPlaceResult" :key="result"
                            class="my-auto"  @click="showInfo(result.latitude, result.longitude)">
                            <th class="align-middle" scope="row">
                                <p class="text-light">
                                    {{ result.name }}
                                </p>
                            </th>
                            <td class="align-middle text-light">
                                <img :src='"https://hatscripts.github.io/circle-flags/flags/" + result.country_code.toLowerCase() + ".svg"' width="48">
                                {{ result.country }}
                            </td>
                        </tr>
                    </tbody>
                </table>
                <!-- weather data of the region -->
                <div class="my-bg" v-show="showFullResult">
                    <h3 class="ps-3 pt-3 text-light">Temperature [°C]</h3>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        height="20%"
                        :options="temperatureResult.options"
                        :series="temperatureResult.series"
                    >
                    </apexchart>
                    <h3 class="ps-3 pt-3 text-light">Precipitation [mm]</h3>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        height="20%"
                        :options="precipitationResult.options"
                        :series="precipitationResult.series"
                    >
                    </apexchart>
                    <h3 class="ps-3 pt-3 text-light">Cloudcover [%]</h3>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        height="20%"
                        :options="cloudyResult.options"
                        :series="cloudyResult.series"
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
        const temperatureResult = ref<any>({
            options:{
                colors: ['#fcba03'],

                chart:{
                    id:"temperature-chart",
                    type:"area",
                    foreColor: "#ccc",

                },
                xaxis:{
                    type: 'datetime',
                    tickAmount: 7,
                    legend: {
                        show: true
                    }
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
                    color: "#00184f",
                },
                fill: {
                    type: 'gradient',
                    gradient:{
                        opacityFrom: 0.9,
                        opacityTo: 0.2,
                    }
                },
            },
            series:[
                {
                    name:"Temperature",
                    data:[],
                }
            ],  
        })
        const precipitationResult = ref<any>({
            options:{
                colors: ['#0818c7'],

                chart:{
                    id:"precipitation-chart",
                    type:"area",
                    foreColor: "#ccc",

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
                    name:"Precipitation",
                    tickAmount: 3
                },
                stroke: {
                    curve: 'smooth',
                    color: "#00184f",
                },
                fill: {
                    type: 'gradient',
                    gradient:{
                        opacityFrom: 0.9,
                        opacityTo: 0.2,
                    }
                },
            },
            series:[
                {
                    name:"Precipitation",
                    data:[],
                }
            ],  
        })
        const cloudyResult = ref<any>({
            options:{
                colors: ['#b3b5c9'],

                chart:{
                    id:"cloudy-chart",
                    type:"area",
                    foreColor: "#ccc",

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
                    name:"Cloudcover",
                    tickAmount: 3
                },
                stroke: {
                    curve: 'smooth',
                    color: "#00184f",
                },
                fill: {
                    type: 'gradient',
                    gradient:{
                        opacityFrom: 0.9,
                        opacityTo: 0.2,
                    }
                },
            },
            series:[
                {
                    name:"Cloudcover",
                    data:[],
                }
            ],  
        })
        const showFullResult = ref<boolean>(false)
        const showPlaceResult = ref<boolean>(false)

        return{
            searchText,
            searchPlaceResult,
            temperatureResult,
            precipitationResult,
            cloudyResult,
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
                this.temperatureResult.series[0].data = []
                this.precipitationResult.series[0].data = []
                this.cloudyResult.series[0].data = []
                this.searchPlaceResult = {}
            }
        },
    },
    methods:{
        async showInfo(longitude:number, latitude:number){
            this.showFullResult = true
            const response = await fetch("https://api.open-meteo.com/v1/forecast?latitude=" + latitude + "&longitude=" + longitude + "&hourly=temperature_2m,precipitation,cloudcover")
            const data = await response.json()
            //and time an temperature to array of object pairs
            data.hourly.time.forEach((time:number, index:number) => {
                console.log(data.hourly.temperature_2m[index])
                this.temperatureResult.series[0].data = [...this.temperatureResult.series[0].data,
                                                            {  x:time, 
                                                               y:data.hourly.temperature_2m[index]
                                                            }
                                                          ]
                this.precipitationResult.series[0].data = [...this.precipitationResult.series[0].data,
                                                            {  x:time, 
                                                               y:data.hourly.precipitation[index]
                                                            }
                                                          ]
                this.cloudyResult.series[0].data = [...this.cloudyResult.series[0].data,
                                                            {  x:time, 
                                                               y:data.hourly.cloudcover[index]
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
    .my-bg{
        background-color: #000000b0;
    }
    thead{
        background:#000000ab;
    }
</style>