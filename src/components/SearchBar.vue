<template>
    <div class="container">
        <div :class="showFullResult ? 'pt-res' : ''" id="search-bar" class="row">
            <div class="col-md-7 mx-auto">
                <div class="container">
                    <div v-show="showFullResult" class="row mx-auto">
                        <div class="col-md-6">
                            <p class="text-center text-light my-labels">Pressure</p>
                            <apexchart
                            :options="actualPressure.options"
                            :series="actualPressure.series"
                            width="100%"
                            >
                            </apexchart>
                        </div>
                        <div class="col-md-6">
                            <p class="text-center text-light my-labels">Wind speed</p>
                            <apexchart
                            :options="actualWind.options"
                            :series="actualWind.series"
                            width="100%"
                            > 
                            </apexchart>
                        </div>
                    </div>
                </div>
                
                <div class="input-group">
                    <input type="text" class="form-control" v-model="searchText" placeholder="Search for...">
                </div>
            </div>
        </div>
        <div id="search-result" class="row">
            <div class="col-md-7 mx-auto">
                <!-- temp search result list -->
                <!-- make this table changing height smooth -->             
                <table id="searchTable" v-show="!showFullResult && searchText != ''" class="my-bg table table-hover table-striped">
                    <thead class="text-light">
                        <tr>
                            <th class="col-6" scope="col">City</th>
                            <th class="col-6" scope="col">Country</th>
                        </tr>
                    </thead>
                    <tbody class="rounded-bottom">
                        <tr v-for="result in searchPlaceResult" :key="result"
                            class="my-auto"  @click="showInfo(result.latitude, result.longitude, result.name)">
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
                <div id="weatherResult" class="my-bg overflow-auto" v-show="showFullResult">
                    <h3 class="ps-3 pt-3 text-light">Temperature [°C]</h3>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        height="220vh"
                        :options="temperatureResult.options"
                        :series="temperatureResult.series"
                    >
                    </apexchart>
                    <h3 class="ps-3 pt-3 text-light">Precipitation [mm]</h3>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        height="220vh"
                        :options="precipitationResult.options"
                        :series="precipitationResult.series"
                    >
                    </apexchart>
                    <h3 class="ps-3 pt-3 text-light">Cloudcover [%]</h3>
                    <apexchart
                        class="pe-2"
                        width="100%"
                        height="220vh"
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
                    tickAmount: 3,
                    min: 0,
                    max: 100,
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
        const actualPressure = ref<any>({
            series: [0],
            options:{
                //set center half-circle background color
                chart: {
                    type: 'radialBar',
                    offsetY: -10,
                    sparkline: {
                        enabled: true
                    }
                },
                plotOptions: {
                    radialBar: {
                        startAngle:-90,
                        endAngle: 90,
                        hollow: {
                            offsetY: -5,
                            size: "50%",
                            background: "#000000ab"
                        },
                        track: {
                            background: "#e7e7e7",
                            strokeWidth: '97%',
                            margin: 5, // margin is in pixels
                            dropShadow: {
                                enabled: true,
                                top: 2,
                                left: 0,
                                color: '#999',
                                opacity: 1,
                                blur: 2
                            }
                        },
                        dataLabels: {
                            name: {
                                show: false
                            },
                            value: {
                                offsetY: -10,
                                fontSize: '22px',
                                color: '#ccc',
                                formatter: function (val:number) {
                                    return val*(1081.3-860)/100 + 860 + " hPa";
                                }
                            },
                        }
                    }
                },
                grid: {
                    padding: {
                        top: -10,
                        bottom: 15,
                    }
                },
                fill: {
                    type: 'gradient',
                    gradient: {
                        shade: 'dark',
                        type: 'vertical',
                        shadeIntensity: 0.5,
                        gradientToColors: ['#00f2ff'],
                        inverseColors: true,
                        opacityFrom: 1,
                        opacityTo: 0.5,
                        stops: [0, 90,100]
                    }
                },
                labels: ['Average Results'],

            },
        })
        const actualWind = ref<any>({
            series: [0],
            options:{
                chart: {
                    type: 'radialBar',
                    offsetY: -10,
                    sparkline: {
                        enabled: true
                    }
                },
                plotOptions: {
                    radialBar: {
                        startAngle:-90,
                        endAngle: 90,
                        hollow: {
                            offsetY: -5,
                            size: "50%",
                            background: "#000000ab"
                        },
                        track: {
                            background: "#e7e7e7",
                            strokeWidth: '97%',
                            margin: 5, // margin is in pixels
                            dropShadow: {
                                enabled: true,
                                top: 2,
                                left: 0,
                                color: '#999',
                                opacity: 1,
                                blur: 2
                            }
                        },
                        dataLabels: {
                            name: {
                                show: false
                            },
                            value: {
                                offsetY: -10,
                                fontSize: '22px',
                                color: '#ccc',
                                formatter: function (val:number) {
                                    //set value to only 2 decimal places
                                    return (val*118/100).toFixed(2) + " km/h";
                                }
                            },
                        }
                    }
                },
                grid: {
                    padding: {
                        top: -10,
                        bottom: 15,
                    }
                },
                fill: {
                    type: 'gradient',
                    gradient: {
                        shade: 'dark',
                        type: 'vertical',
                        shadeIntensity: 0.5,
                        gradientToColors: ['#2F4858'],
                        inverseColors: true,
                        opacityFrom: 1,
                        opacityTo: 0.5,
                        stops: [0, 90, 100]
                    }
                },
                labels: ['Average Results'],

            },
        })
        //controling variables
        const showFullResult = ref<boolean>(false)
        const showPlaceResult = ref<boolean>(false)
        const searching = ref<boolean>(true)

        const tdHeight = ref<number>(40)
        const tableCellHeight = ref<number>(65)
        const prewResultsCount = ref<number>(0)
        //color constants
        const wind = ref({
            speed:{
                max: 118,
            },
            colorAngle:{
                max: 112,
                min: 0,
            }
        })
        const pressure = ref({
            max: 1081.3,
            min: 860,
            colorAngle:{
                max: 248,
                min: 0,
            }
        })
        return{
            searchText,
            tdHeight,
            tableCellHeight,
            prewResultsCount,
            //forecast variables
            searchPlaceResult,
            temperatureResult,
            precipitationResult,
            cloudyResult,
            //actual variables
            actualWind,
            actualPressure,
            //logic variables
            showFullResult,
            showPlaceResult,
            searching,
            //properties settings
            wind,
            pressure,
        }
    },
    components: {
        apexchart: VueApexCharts,
    },
    watch: {
        async searchText(newVal){
            if(newVal != '' && this.searching){
                let table:any = document.getElementById('searchTable')
                let prewResultCounter = 0
                

                this.showFullResult = false
                this.showPlaceResult = false
                this.temperatureResult.series[0].data = []
                this.precipitationResult.series[0].data = []
                this.cloudyResult.series[0].data = []
                //this.searchPlaceResult = {}
                //make smooth height change animation to table when searching
                this.showFullResult = false
                const response = await fetch("https://geocoding-api.open-meteo.com/v1/search?name=" + newVal)
                const data = await response.json()
                if(data.results){
                    this.showPlaceResult = true
                    if(data.results.length > 5){
                        //show new render after old is updated
                        this.searching = false
                        this.searchPlaceResult = data.results.slice(0, 5)
                        this.setHeight(table, 5, prewResultCounter)
                        prewResultCounter = 5
                    }
                    else{
                        this.searchPlaceResult = data.results
                        this.setHeight(table, data.results.length, prewResultCounter)
                        prewResultCounter = data.results.length
                    }
                }
                else{
                    this.showPlaceResult = false
                    this.searchPlaceResult = {}
                    table.style.height = "0px"
                }
            }
            if(!this.searching){
                this.searching = true
            }
        },
    },
    methods:{
        setHeight(table:any, newNum:number, oldNum:number){
            if(newNum != oldNum){
                if(newNum > oldNum){
                    let tempHeight = 0
                    for(let i = 0; i < newNum * this.tableCellHeight - oldNum * this.tableCellHeight; i++){
                        setTimeout(() => {
                            tempHeight++
                            table.style.height = this.tdHeight + tempHeight + "px"
                        }, 10)
                    }
                }
            }
        },
        hslToHex(h:number, s:number, l:number) {
            l /= 100;
            const a = s * Math.min(l, 1 - l) / 100;
            const f = (n: number) => {
                const k = (n + h / 30) % 12;
                const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
                return Math.round(255 * color).toString(16).padStart(2, '0');   // convert to Hex and prefix "0" if needed
            };
            return `#${f(0)}${f(8)}${f(4)}`;
        },
        async showInfo(longitude:number, latitude:number, name:string){
            this.showFullResult = true
            this.searching = false
            this.searchText = name
            const response = await fetch("https://api.open-meteo.com/v1/forecast?latitude=" + latitude + "&longitude=" + longitude + "&hourly=temperature_2m,precipitation,cloudcover,surface_pressure,windspeed_10m,winddirection_10m")
            const data = await response.json()

            //set color of gradient by pressure ratio
            this.actualPressure.series = [100*(data.hourly.surface_pressure[0]-this.pressure.min)/ (this.pressure.max-this.pressure.min)]
            let pressureColorAngle = this.pressure.colorAngle.max*(1 - (data.hourly.surface_pressure[0]-this.pressure.min)/(this.pressure.max - this.pressure.min))
            let pressureHexColor = this.hslToHex(pressureColorAngle, 98, 50)
        
            this.actualPressure.options.fill.gradient.gradientToColors.pop()
            this.actualPressure.options.fill.gradient.gradientToColors.push(pressureHexColor)

            //set color of gradient by wind ratio
            this.actualWind.series = [100*data.hourly.windspeed_10m[0]/this.wind.speed.max]
            let windColorAngle = (this.wind.colorAngle.max / Math.sqrt(this.wind.colorAngle.max))*Math.sqrt(this.wind.colorAngle.max*(1 - data.hourly.windspeed_10m[0]/this.wind.speed.max))
            let windHexColor = this.hslToHex(windColorAngle, 98, 50)
            
            this.actualWind.options.fill.gradient.gradientToColors.pop()
            this.actualWind.options.fill.gradient.gradientToColors.push(windHexColor)
             
            //and time an temperature to array of object pairs
            data.hourly.time.forEach((time:number, index:number) => {
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
    #searchTable{
        height: 0;
        overflow: hidden;
    }
    #weatherResult{
        height: 40%;
    }
    tbody tr:hover{
        background-color: #010035d0;
        transition: .3s;
        cursor: pointer
    }
    .my-bg{
        background-color: #000000b0;
        transition: .3s;
    }
    .my-labels{
        position: relative;
        z-index: 1;
        top:80%;
        color:#ccc;
    }
    .pt-res{
        padding-top: 65vh;
    }
    thead{
        background:#000000ab;
    }
    /* ===== Scrollbar CSS ===== */
    /* Firefox */
    * {
        scrollbar-width: auto;
    }

    /* Chrome, Edge, and Safari */
    *::-webkit-scrollbar {
        width: 16px;
    }

    *::-webkit-scrollbar-track {
        background: rgba(0, 0, 0, 0.219);
    }

    *::-webkit-scrollbar-thumb {
        background-color: rgba(23, 22, 24, 0.643);
        border-radius: 10px;
    }
</style>