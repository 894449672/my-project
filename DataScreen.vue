<template >
    <div id="app-content">
        <div class="containerBox">
            <!-- 左边区域 -->
            <div class="left">
                <!-- 车辆情况 -->
                <div class="item">
                    <div class="tabItem">
                        <p class="tabItemTitle">车辆情况</p>
                        <div class="tabItemConent">
                            <div class="totalCarNum" style="margin:10px;">
                                <span>车辆总数</span>
                                <p class="carNum">
                                    <span v-for='(item,index) in vehicleData.totalCar' class="span1"><i>{{item}}</i></span> 
                                </p>
                                <span>辆</span>
                            </div>
                            <div class="echartItem" id="vehicle" style="height: calc(100% - 50px)"></div>
                        </div>
                    </div>
                </div> 
                <!-- 历史数据 -->
                <div class="item">
                    <div class="tabItem">
                        <p class="tabItemTitle">历史数据</p>
                        <div class="tabItemConent" >
                            <div class="history">
                                <div class="historyItem bc1" >
                                    <p class="historyName">最多车辆</p>
                                    <p class="historyData">{{historyData.historyVehiclesMax}}</p>
                                </div>
                                <div class="historyItem bc2">
                                    <p class="historyName">总里程数(km)</p>
                                    <p class="historyData">{{historyData.sumMileage}}</p>
                                </div>
                                <div class="historyItem bc3">
                                    <p class="historyName">平均故障率</p>
                                    <p class="historyData">{{historyData.breakdown}}%</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div> 
                <!-- 城市维修TOP5 -->
                <div class="item">
                    <div class="tabItem">
                        <p class="tabItemTitle">城市维修TOP5</p>
                        <div class="tabItemConent">
                            <div class="echartItem" id="cityFixed"></div>
                        </div>
                    </div>
                </div> 
            </div>
            <!-- 中间区域 -->
            <div class="middle">
                <!-- 车辆分布 -->
                <div class="item" style="height: 60%;" >
                    <div class="tabItem">
                        <p class="tabItemTitle">车辆分布</p>
                        <div class="tabItemConent">
                            <div class="echartItem" id="map"></div>
                        </div>
                    </div>
                </div>
                <!-- 故障维修统计 -->
                <div class="item" style="height: calc(40% - 42px);">
                    <div class="tabItem">
                        <p class="tabItemTitle">故障维修统计</p>
                        <div class="tabItemConent">
                            <div class="echartItem" id='faultLine'></div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- 右边区域 -->
            <div class="right">
                <!-- 故障比例 -->
                <div class="item" style=" height: 30%">
                    <div class="tabItem">
                        <p class="tabItemTitle">故障比例</p>
                        <div class="tabItemConent">
                            <div class="echartItem" id="fault"></div>
                        </div>
                    </div>
                </div>
                <!-- 故障分布城市 -->
                <div class="item" style=" height: 30%">
                    <div class="tabItem">
                        <p class="tabItemTitle">故障分布城市</p>
                        <div class="tabItemConent">
                            <div class="echartItem" id="city"></div>
                        </div>
                    </div>
                </div>
                <!-- 故障报警 -->
                <div class="item" style=" height: calc(40% - 60px)">
                    <div class="tabItem">
                        <p class="tabItemTitle">故障报警</p>
                        <div class="tabItemConent">
                            <div class="echartItem faultWarningItem">
                            <div class="itemHeader">
                                <span>车牌号</span>
                                <span>故障ID</span>
                                <span>故障类型</span>
                                <span>故障等级</span>
                            </div>
                            <div class="itemBody">
                                <div class="bodyItem" v-for='(item,index) in faultWarningData' :key='index'>
                                    <span v-for='(item1,index1) in item.data'>{{item1}}</span>
                                </div>
                            </div>
                        </div>
                        </div>
                    </div>
                </div>
            </div>
      </div>
  </div>
</template>

<script>
//   import 'echarts/map/js/china.js';
  export default {
    data () {
      return{
        vehicleData: {
            totalCar: [0, 0, 0, 0, 0 , 0],
        },
        // 故障报警数据
        faultWarningData:[],
        provincesta:[],
        // active:flase,
        tableData:[],
        dialogTableVisible: false,
        dialogFormVisible: false,
        form:{
          UserName:'',
          Password:'',
          RePassword:'',
          Telephone:'',
          RealName:'',
          Company:'',
          Dept:'',
        },
        rules:{
          UserName:[
            { required: true, message: "请输入用户名", trigger: "blur"},
            { pattern: /^[a-zA-Z0-9]{6,16}$/, message:'用户名格式错误', trigger: "blur"}
          ],
          Password:[
            { required: true, message: "请输入密码",trigger: "blur"},
            { pattern: /[a-zA-Z\d+]{6,16}/, message:'密码格式错误',trigger:'blur'}
          ],
          RePassword:[
            { required: true, message: "请输入确认密码", trigger: "blur"},
            // { validator: pwdCheckValidate, trigger: "blur"}
          ],
          Telephone:[
            { required: true, message: "请输入电话号码", trigger: "blur"},
            { pattern: /^1[3|4|5|7|8][0-9]{9}$/, message: "电话号码格式错误", trigger: "blur"}
          ],
          RealName:[
            { required: true, message: "请输入真实姓名", trigger: "blur"},
            { pattern: /^[A-Za-z0-9\u4e00-\u9fa5]{1,16}$/, message: "真实姓名格式错误", trigger: "blur"}
          ],
          Company:[
            { required: true, message: '请选择关联运营公司', trigger: 'change' },
          ],
          Dept:[
            { required: true, message: '请选择关联部门', trigger: 'change' },
          ],
        },
        historyData:{
            historyVehiclesMax:0,
            sumMileage:0,
            breakdown:0
        }
      }
    },
    created(){
        this.axios('static/json/faultWarningData.json').then(res => {
            this.faultWarningData = res.data.data
        }).catch(err => {
            console.log(err)
        }) 
        this.axios('static/json/provincesta.json').then(res => {
            var d = res.data.totalCar;
            var totalCar = d.toString().split('');
            this.vehicleData.totalCar.splice(-totalCar.length);
            this.vehicleData.totalCar = this.vehicleData.totalCar.concat(totalCar);
        }).catch(err => {
            console.log(err)
        }) 
        this.axios('static/json/history.json').then(res => {
            this.historyData.historyVehiclesMax = res.data.historyVehiclesMax
            this.historyData.sumMileage = res.data.sumMileage
            this.historyData.breakdown = res.data.breakdown
        }).catch(err => {
            console.log(err)
        }) 
    },
    mounted() {
        //车辆分布
        this.drawMap()
        // 车辆情况
        this.drawVehicleBar()
         // 车辆维修 top5城市
        this.drawCityFixed();
        // 故障维修统计
        this.drawFaultLine()
        // 故障比例
        this.drawFaultPie();
        // 车辆城市分布（或故障比例）
         this.drawCityCarBar1();
         // 故障报警滚动
        this.faultAlarm();
    },
    methods: {
        //城市分布--地图
        drawMap() {
            var echart = this.$echarts.init(document.getElementById("map"));
            let option = {
                tooltip: {
                    trigger: 'item',
                    formatter: '{b}:{c}'
                },
                visualMap: {
                    min: 0,
                    max: 20,
                    left: 'left',
                    top: 'bottom',
                    text: ['高', '低'],
                    calculable: true,
                    inRange: {
                        color: ['#ffffff', '#E0DAFF', '#ADBFFF', '#9CB4FF', '#6A9DFF', '#3889FF']
                    },
                    textStyle:{
                        color:"#fff"
                    }
                },
                series: [
                    {
                        zlevel: 1,
                        type: 'map',
                        mapType: 'china',
                        selectedMode : 'multiple',
                        roam: true,
                        left: "10%",
                        right: '10%',
                        label: {
                            normal: {
                                show: true
                            },
                            emphasis: {
                                show: true
                            }
                        },
                        data:[]
                    }
                ]
            };
            echart.setOption(option);
            window.addEventListener("resize", () => { echart.resize();});
            this.axios('static/json/provincesta.json').then(res => {
                echart.setOption({
                    series:[{
                        data:res.data.data
                    }]
                })
            }).catch(err => {
                console.log(err)
            }) 
        },
        //车辆情况 --- 柱状图
        drawVehicleBar(){
            let labelData=[17, 11, 86 ,90 ]; // 右侧显示数据
            let echart = this.$echarts.init(
                document.getElementById("vehicle")
            );
            let option = {
                grid: {
                    left: 30,
                    right: '30%',
                    bottom: 10,
                    top: 0,
                    containLabel: true
                },
                tooltip: {
                    show: "true",
                    trigger: "axis",
                    axisPointer: {
                        // 坐标轴指示器，坐标轴触发有效
                        type: "shadow" // 默认为直线，可选为：'line' | 'shadow'
                    }
                },
                xAxis: {
                    type: "value",
                    axisTick: { show: false },
                    // X轴的字体样式
                    axisLabel: {
                        show: false,
                        textStyle: {
                            color: "#abfeff",
                            fontSize: "14"
                        }
                    },
                    axisLine: {
                        show: false,
                        lineStyle: {
                            color: "#4064b4"
                        }
                    },
                    splitLine: {
                        show: false
                    }
                },
                yAxis: [
                    {
                        type: "category",
                        axisTick: { show: false },
                        axisLine: {
                            show: false,
                            lineStyle: {
                                color: "#4064b4"
                            }
                        },
                        axisLabel: {
                            interval: 0,
                            textStyle: {
                                color: "#abfeff",
                                fontSize: 14
                            }
                        },
                        data: []
                    },
                    {
                        type: "category",
                        axisLine: { show: false },
                        axisTick: { show: false },
                        axisLabel: { show: false },
                        splitArea: { show: false },
                        splitLine: { show: false },
                        data: []
                    }
                ],
                series: [
                    {
                        name: "总数",
                        type: "bar",
                        yAxisIndex: 1,
                        barWidth:12, 
                        label:{
                            normal:{
                                show:true,
                                position:'right',
                                color:'#abfeff',
                                fontSize:14,
                                formatter:(data)=>{
                                    let index=data.dataIndex
                                    var data1= [16, 10, 86 ,90 ]
                                    return (Math.ceil((data1[index]/data.value)*100)+'%'+'       '+labelData[index]+'(辆)')
                                }
                            }
                        },
                        itemStyle: {
                            normal: {
                                show: true,
                                color: "#3356a1",
                                barBorderRadius: 7,
                                borderWidth: 0,
                                borderColor: "#333"
                            }
                        },
                        barGap: "0%",
                        barCategoryGap: "50%",
                        data: []
                    },
                    {
                        name: "类别",
                        type: "bar",
                        barWidth:12, 
                        label:{
                            normal:{
                                show:false,
                                formatter:(data)=>{
                                    labelData.push(((data.value/97)*100).toFixed(0)+'%')
                                }
                            }
                        },
                        itemStyle: {
                            normal: {
                                show: true,
                                color: "#5de3e1",
                                barBorderRadius: 7,
                                borderWidth: 0,
                                borderColor: "#333",
                                color:(params)=>{
                                    var colorList=[
                                        ['#b7d1e0','#7e95bc'],
                                        ['#ffbce8','#f97186'],
                                        ['#78edfd','#3f87ff'],
                                        ['#96f8bd','#41e4d4'],
                                    ];
                                    var index=params.dataIndex;
                                    if(params.dataIndex>=colorList.length){
                                        index=params.dataIndex-colorList.length
                                    }
                                    return new this.$echarts.graphic.LinearGradient(0,0,1,0,
                                    [
                                        {offset:0,color:colorList[index][0]},
                                        {offset:1,color:colorList[index][1]},
                                    ])
                                }
                            }
                        },
                        barGap: "0%",
                        barCategoryGap: "50%",
                        data: []
                    }
                ]
            };
            echart.setOption(option);
            this.axios('static/json/vehicleState.json').then(res => {
                echart.setOption({
                    yAxis:[{
                        data: res.data.status
                    },{
                        data: res.data.status
                    }],
                    series:[{
                        data:res.data.sum
                    },{
                        data:res.data.data
                    }]
                })
            }).catch(err => {
                console.log(err)
            }) 
            window.addEventListener("resize", () => { echart.resize();});
        },
        // 车辆维修分布 top5 --- 柱状图
        drawCityFixed(){
            let echart = this.$echarts.init( document.getElementById("cityFixed"));
            let option = {
                legend: {
                    icon: "circle",
                    top: 0,
                    right: 0,
                    textStyle: {
                        color: "#fff",
                        fontSize: 12
                    },
                    data: ["维修量", "报修量"]
                },
                grid: {
                    left: "3%",
                    right: 20,
                    bottom: 10,
                    top: 20,
                    containLabel: true
                },
                tooltip: {
                    show: "true",
                    trigger: "axis",
                    axisPointer: {
                        // 坐标轴指示器，坐标轴触发有效
                        type: "shadow" // 默认为直线，可选为：'line' | 'shadow'
                    }
                },
                xAxis: {
                    type: "value",
                    axisTick: { show: false },
                    // X轴的字体样式
                    axisLabel: {
                        show: true,
                        textStyle: {
                            color: "#abfeff",
                            fontSize: "14"
                        }
                    },
                    axisLine: {
                        show: true,
                        lineStyle: {
                            color: "#4064b4"
                        }
                    },
                    splitLine: {
                        show: false
                    }
                },
                yAxis: [
                    {
                        type: "category",
                        axisTick: { show: false },
                        axisLine: {
                            show: true,
                            lineStyle: {
                                color: "#4064b4"
                            }
                        },
                        axisLabel: {
                            interval: 0,
                            textStyle: {
                                color: "#abfeff",
                                fontSize: 14
                            }
                        },
                        data: []
                    },
                    {
                        type: "category",
                        axisLine: { show: false },
                        axisTick: { show: false },
                        axisLabel: { show: false },
                        splitArea: { show: false },
                        splitLine: { show: false },
                        data: []
                    }
                ],
                series: [
                    {
                        name: "报修量",
                        type: "bar",
                        barWidth:14, 
                        yAxisIndex: 1,
                        itemStyle: {
                            normal: {
                                show: true,
                                color: "#277ace",
                                barBorderRadius: 50,
                                borderWidth: 0,
                                borderColor: "#333"
                            }
                        },
                        barGap: "0%",
                        barCategoryGap: "50%",
                        data: []
                    },
                    {
                        name: "维修量",
                        type: "bar",
                        barWidth:14, 
                        itemStyle: {
                            normal: {
                                show: true,
                                color: "#5de3e1",
                                barBorderRadius: 0,
                                borderWidth: 0,
                                borderColor: "#333"
                            }
                        },
                        barGap: "0%",
                        barCategoryGap: "50%",
                        data: []
                    }
                ]
            };
            echart.setOption(option);
            window.addEventListener("resize", () => { echart.resize();});
            this.axios('static/json/repairCityTop.json').then(res => {
                echart.setOption({
                    yAxis:[{
                        data: res.data.city
                    },{
                        data: res.data.city
                    }],
                    series:[{
                        data:res.data.repairNum
                    },{
                        data:res.data.maintenanceNum
                    }]
                })
            }).catch(err => {
                console.log(err)
            }) 
        },
        // 故障维修统计 --- 折线图
        drawFaultLine(){
            let echart= this.$echarts.init(document.getElementById("faultLine"));     
            let option={
                color:['#08f9e5','#3b76f5','#e74f4f'],
                legend: {
                    icon: "circle",
                    top: 0,
                    right: 0,
                    textStyle: {
                        color: "#abfeff",
                        fontSize: 12
                    },
                    data:['维修次数','报修次数','故障次数']
                },
                xAxis: {
                    type: 'category',
                    boundaryGap: false,
                    data: [],
                    // X轴的字体样式
                    axisLabel: {        
                        show: true,
                        textStyle: {
                            color: '#abfeff',
                            fontSize:'14'
                        }
                    },
                    // X轴线样式
                    axisLine:{
                        lineStyle:{
                            color:'#4064b4'
                        }
                    },
                    // 横向网格线
                    splitLine:{
                        show:false,
                    }                
                },
                yAxis: {
                    type: 'value',
                    // Y轴的字体样式
                    axisLabel: {        
                        show: true,
                        textStyle: {
                            color: '#abfeff',
                            fontSize:'14'
                        }
                    },
                    // Y轴线样式
                    axisLine:{
                        lineStyle:{
                            color:'#4064b4'
                        }
                    },
                    // 横向网格线
                    splitLine:{
                        show:true,
                        lineStyle:{
                            color:'#264487'
                        }
                    }
                },
                series: [
                    {
                        name:'维修次数',
                        type: 'line',
                        data: [],
                        smooth: true,
                        symbol: "none",  //拐角为实心,
                        itemStyle:{
                            normal:{
                                // 折现样式
                                lineStyle:{
                                    width:2,
                                    color:'#08f9e5',
                                },
                            }
                        },                        
                        areaStyle: {
                            color: new this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                                offset: 0,
                                color: 'rgba(8, 249, 229,0.8)'
                            },{
                                offset: 0.8,
                                color: 'rgba(8, 249, 229,0.03)'
                            }])
                        },
                    },
                    {
                        name:'报修次数',
                        type: 'line',
                        data: [],
                        smooth: true,
                        symbol: "none",  //拐角为实心,
                        itemStyle:{
                            normal:{
                                // 折现样式
                                lineStyle:{
                                    width:2,
                                    color:'#3b76f5',
                                },
                            }
                        },                        
                        areaStyle: {
                            color: new this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                                offset: 0,
                                color: 'rgba(51, 86, 161,0.8)'
                            },{
                                offset: 0.8,
                                color: 'rgba(51, 86, 161,0.03)'
                            }])
                        },
                    },
                    {
                        name:'故障次数',
                        type: 'line',
                        data: [],
                        smooth: true,
                        symbol: "none",  //拐角为实心,
                        itemStyle:{
                            normal:{
                                // 折现样式
                                lineStyle:{
                                    width:2,
                                    color:'#e74f4f',
                                },
                            }
                        },
                        areaStyle: {
                            color: new this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                                offset: 0,
                                color: 'rgba(231, 79, 79,0.8)'
                            },{
                                offset: 0.8,
                                color: 'rgba(231, 79, 79,0.03)'
                            }])
                        },
                    }
                ],
                grid: {
                    left: "3%",
                    right: 20,
                    bottom: 10,
                    top: 24,
                    containLabel: true
                },
            };
            echart.setOption(option)    
            window.addEventListener("resize", () => { echart.resize();});   
            this.axios('static/json/repairCategory.json').then(res => {
                echart.setOption({
                    xAxis:{
                        data:res.data.category
                    },
                    series:[{
                        data:res.data.maintenanceNum
                    },{
                        data:res.data.repairNum
                    },{
                        data:res.data.failureNum
                    }]
                })
            }).catch(err => {
                console.log(err)
            }) 
        },
        // 得到故障比例统计数据 --- 饼图
        drawFaultPie() {
            let echart = this.$echarts.init(document.getElementById("fault"));
            this.axios('static/json/failureRatio.json').then(res => {
                let data = res.data.data2;
                let total = 0; // 总和
                for (let v of data) {
                    total += v.value;
                }
                let option = {
                tooltip: {
                    trigger: "item",
                    show: true
                },
                legend: [
                    {
                        icon: "circle",
                        orient: "vertical",
                        top: '20%',
                        right: '10%',
                        align: "left",
                        data: ["电池组", "电机", "发动机", "其他"],
                        formatter: name => {
                            for (let i in data) {
                                if (name == data[i].name) {
                                    return ( name +" " +((data[i].value / total) * 100).toFixed(0) +"%");
                                }
                            }
                        },
                        textStyle: {
                            fontSize: 14,
                            color: "#fff"
                        }
                    }
                ],
                series: [
                    {
                        type: "pie",
                        radius: ["50%", "70%"],
                        center: ["32%", "50%"],
                        avoidLabelOverlap: false,
                        label: {
                            normal: {
                                show: false,
                                textStyle: {
                                    color: "#abfeff"
                                }
                            }
                        },
                        labelLine: {
                            show: true,
                            length: 20 // 改变标示线的长度
                        },
                        data: data,
                        color: ["#298bfb", "#d45df3", "#fe624b", "#04f7ae"],
                        itemStyle: {
                            normal: {
                                borderWidth: 4,
                                borderColor: "#0c2458",
                                label: {
                                    formatter: data => {
                                        return (((data.value / total) *100).toFixed(0) +"%" +"\n \n" +data.name);
                                    }
                                }
                            }
                        }
                    }
                ]
            };
            
            echart.setOption(option);
            window.addEventListener("resize", () => { echart.resize();});   
            }).catch(err => {
                console.log(err)
            }) 
        },
        // 车辆城市分布 top5（或故障比例） --- 柱状图
        drawCityCarBar1(){
            let echart = this.$echarts.init(document.getElementById("city"));
            let option = {
                xAxis: {
                    type: "category",
                    data: [],
                    // X轴的字体样式
                    axisLabel: {
                        show: true,
                        textStyle: {
                            color: "#abfeff",
                            fontSize: "14"
                        }
                    },
                    // X轴线样式
                    axisLine: {
                        lineStyle: {
                            color: "#4064b4"
                        }
                    }
                },
                yAxis: {
                    type: "value",
                    // Y轴的字体样式
                    axisLabel: {
                        show: true,
                        textStyle: {
                            color: "#abfeff",
                            fontSize: "14"
                        }
                    },
                    // Y轴线样式
                    axisLine: {
                        lineStyle: {
                            color: "#4064b4"
                        }
                    },
                    // 横向网格线
                    splitLine: {
                        show: false
                    }
                },
                series: [
                    {
                        type: "bar",
                        barWidth: 30,
                        data: [],
                        itemStyle: {
                            normal: {
                                // 柱状图颜色是渐变色
                                color: new this.$echarts.graphic.LinearGradient(
                                    0,
                                    0,
                                    0,
                                    1,
                                    [
                                        {
                                            offset: 0,
                                            color: "#68f4ab"
                                        },
                                        {
                                            offset: 1,
                                            color: "#30a99e"
                                        }
                                    ]
                                )
                            }
                        },
                        label: {
                            normal:{
                                show:false,
                                position:'top',
                                color:'#abfeff',
                                fontSize:14
                            }
                        }
                    }
                ],
                grid: {
                    left: "3%",
                    right: 20,
                    bottom: 10,
                    top: '4%',
                    containLabel: true
                }
            };
            echart.setOption(option);
            window.addEventListener("resize", () => { echart.resize();});   
            this.axios('static/json/failureCity.json').then(res => {
                echart.setOption({
                    xAxis:{
                        data:res.data.city
                    },
                    series:[{
                        data:res.data.count
                    }]
                })
            }).catch(err => {
                console.log(err)
            }) 
        },
        // 故障报警滚动
        faultAlarm(){
            let itemBody=document.getElementsByClassName('itemBody')[0];
            let i=36
            let itemTimer=setInterval(()=>{
                i--
                itemBody.style.top=i+'px'
                if(i==0){
                    i=36
                }
            },80)
        }
    }
}
</script>

<style scoped>
#app-content{
    height: calc(100% - 60px);
    width: 95%;
    margin-left: 4%;
}
.containerBox{
  width: 100%;
  height:100%;
  color: #abfeff;
}
.item{
    box-shadow: 0 2px 7px rgba(77, 145, 255, 0.15);
    width: 100%;
    height: calc(33% - 18px);
    margin-top:18px;
}

.left{
    width: 25%;
    float: left;
    height: 100%;
}
.middle {
    width: calc(50% - 40px);
    margin: 0 20px;
    float: left;
    height: 100%;
}
.right{
    width: 25%;
    float: right;
    height: 100%;
}

/* 车辆情况 */
.totalCarNum {
    display: flex;
    align-items: center;
    padding-left: 30px;
}
.carNum {
    margin: 0 16px;
    display: flex;
}
.span1 {
    display: inline-block;
    width: 30px;
    height: 40px;
    line-height: 40px;
    background: #23459b;
    font-size: 26px;
    text-align: center;
    position: relative;
}
/* 历史数据 */
.history {
    padding: 10px 0px 10px 0px;
    width: 80%;
    height: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}
.historyItem {
    width: 100%;
    flex: 1;
    margin-bottom: 10px;
    padding-left: 50px;
    color: #fff;
}
.bc1{
    background: url(../assets/images/historyBg1_03.png) no-repeat;
        background-size: 100% 100%;
        box-shadow: 0 4px 4px rgba(126, 117, 253, 0.2),
            5px 5px 5px rgba(126, 117, 253, 0.2),
            -5px 4px 4px rgba(126, 117, 253, 0.2);
}
.bc2{
    background: url(../assets/images/historyBg2_03.png) no-repeat;
        background-size: 100% 100%;
        box-shadow: 0 4px 4px rgba(93, 194, 219, 0.2),
            5px 5px 5px rgba(93, 194, 219, 0.2),
            -5px 4px 4px rgba(93, 194, 219, 0.2);
}
.bc3{
    background: url(../assets/images/historyBg3_03.png) no-repeat;
        background-size: 100% 100%;
        box-shadow: 0 4px 4px rgba(238, 109, 110, 0.2),
            5px 5px 5px rgba(238, 109, 110, 0.2),
            -5px 4px 4px rgba(238, 109, 110, 0.2);
}
.echartItem {
    width: 100%;
    height: 100%;
}
/* 故障报警 */
.echartItem.faultWarningItem{
    position: relative;
    overflow: hidden;
}
.itemHeader{
    height: 34px;
    line-height: 34px;
    font-size: 14px;
    background: #1d48a5;
    color: #fff;
    position: absolute;
    width: 100%;
    left: 0;
    top: 0;
    z-index: 1;
}
.itemHeader,.bodyItem{
    display: flex;
}
span{
    width: 25%;
    text-align: center;
}
.itemBody{
    position: absolute;
    top: 34px;
    width: 100%;
}
</style>
