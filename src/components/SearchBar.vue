<template>
    <div class="container">
        <div id="search-bar" class="row">
            <div class="col-md-7 mx-auto">
                <div class="input-group">
                    <input type="text" class="form-control" v-model="searchText" placeholder="Search for...">
                    <span class="input-group-btn">
                        <button class="btn btn-primary" type="button">Go!</button>
                    </span>
                </div>
            </div>
        </div>
        <div :class="(searchText != '') && (searchText.length > 1) && (searchPlaceResult != null)? 'show' : 'hide'" id="search-result" class="row">
            <div class="col-md-7 mx-auto">
                <!-- temp search result list -->
                <table v-show="!showFullResult" class="table table-hover table-dark table-striped">
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
                                    {{result.name}}
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
                <div v-show="showFullResult">
                    <apexchart
                        class="bg-dark pe-2"
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
                title:{
                    text:"Temperature in the region",
                    align:"left",
                    style:{
                        fontSize: "24px",
                        fontFamily: "Helvetica, Arial, sans-serif",
                        fontWeight: "bold",
                        color: "#fff",
                    },
                },
                markers: {
                    size: 5,
                    colors: ["#000524"],
                    strokeColor: "#00BAEC",
                    strokeWidth: 3,
                },
                xaxis:{
                    type: 'category',
                    tickAmount: 7,
                    offsetX: 20,
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
                    name:"Temperature",
                    data:[],
                }
            ],
            
        })
        const showFullResult = ref<boolean>(false)

        return{
            searchText,
            searchPlaceResult,
            searchWeatherResult,
            showFullResult,
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
                this.searchPlaceResult = data.results
                if(data.results.length > 5){
                    this.searchPlaceResult = data.results.slice(0, 5)
                }
            }
            else if(newVal == ''){
                this.showFullResult = false
                this.searchWeatherResult.options.xaxis.categories = []
                this.searchWeatherResult.series[0].data = []
                this.searchPlaceResult = {}
            }
        },
    },
    methods:{
        formatDate(date:number) {
            //return date in format day/HH:MM
            const d = new Date(date)
            //change return format to day, month HH:MM using toLacaeString()
            return d.toLocaleString("en-EN",{ weekday: 'short', month: 'short', hour: 'numeric', minute: 'numeric' })
        },
        async showInfo(longitude:number, latitude:number){
            this.showFullResult = true
            const response = await fetch("https://api.open-meteo.com/v1/forecast?latitude=" + latitude + "&longitude=" + longitude + "&hourly=temperature_2m,relativehumidity_2m,windspeed_10m")
            const data = await response.json()
            //this.searchWeatherResult.series[0].data = data.hourly.temperature_2m
            //this.searchWeatherResult.options.xaxis.categories = data.hourly.time
            //console.log(this.searchWeatherResult.options.xaxis.categories[0])
            //and time an temperature to array of array pairs
            data.hourly.time.forEach((time:number, index:number) => {
                //convert time to day/HH:mm format
                let timeString = this.formatDate(time)
                console.log(data.hourly.temperature_2m[index])
                this.searchWeatherResult.series[0].data.push({  x:this.formatDate(time), 
                                                                y:data.hourly.temperature_2m[index]})
            })
            console.log(this.searchWeatherResult.series[0].data)
        }
    }

})
</script>
<style lang="scss">
    $height: 0%;
    @mixin height($height){
        @if($height:0%){
            animation: fadeOut .5s forwards;
            $height: 100%;
        }
        @else{
            animation: fadeIn .5s forwards;
            $height:0%
        }
    }
   //make search-result smooth appear/disappear when displaying/hiding search results
    #search-result{
        height: $height;
        overflow: hidden;
    }
    .show{
        animation: fadeIn 1s forwards;
    }
    .hide{
        animation: fadeOut 1s forwards;
    }
    @keyframes fadeIn {
        from {
            height: 0%;
        }
        to {
            height: 100%;
        }
    }
    @keyframes fadeOut {
        from{
            height: 100%;
        }
        to {
            height: 0%;
        } 
    }
</style>