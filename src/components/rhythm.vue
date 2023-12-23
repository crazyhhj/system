<template>
    <button @click="showRhythmLine">点击显示</button>
    <button @click="clear">clear</button>
    <div class="container_rhythm">
        <div class="sub_container">
            <div class="chartContainer" ref="rhythmLineArea" id="ss"></div>
            <div class="chartContainer" id="timerect" style="height:40px;"></div>
            <div class="chartContainer" id="chainChart" style="height:200px;"></div>
            <div class="chartContainer" id="chainChart" style="height:200px;">
                {{ showtext }}
            </div>
        </div>
        <div class="sub_container" id="performance_methods" style="width: 300px;"></div>
        <div id="charrr"></div>
    </div>
</template>

<script>
import * as d3 from 'd3'
import * as echarts from 'echarts'

export default {
    name: 'rhythm',
    components: {},
    data() {
       return{
            rhythmDetailData: this.$store.state.rhythmDetailData,
            index:0,
            showtext:'this is initValue',
       }
    },
    mounted() {
    },
    computed: {

    },
    watch: {

    },
    methods: {
        clear(){
            this.$store.commit('clearTrendList');
        },
        showRhythmLine() {
            const that = this;
            // var dom = this.$refs.rhythmLineArea;
            var dom = document.getElementById('ss');
            // var myChart = echarts.init(chartDom);
            var myChart = echarts.init(dom, null, {
                renderer: 'svg',
                useDirtyRect: false
            });
            var option;
            const fakeData = [];

            for (let i = 0; i < 100; i++) {
                let name = `2023-${i}`;
                let baseline = 10 * Math.random();
                let data = {
                    date: name,
                    value: baseline,
                    l: baseline - 3 * Math.random(),
                    u: baseline + 3 * Math.random()
                }
                fakeData.push(data);
            }
            const quickLookData = this.$store.getters.getScheduling;
            // console.log(data[0]);
            // const data = this.rhythmDetailData[index];
            const dataOrg = this.$store.state.rhythmDetailData;
            const showList = this.$store.state.trendList;
            let data = showList.map(d=>dataOrg[d]).flat()
            data.forEach((d,idx)=>{
                d.idx = idx;
            })
            myChart.showLoading();
            function doChart(data) {
                myChart.hideLoading();
                var base = -data.reduce(function (min, val) {
                    return Math.floor(Math.min(min, val.emotion));
                }, Infinity);
                myChart.setOption(
                    (option = {
                        title: {
                            text: 'rhythm_line',
                            subtext: 'Example in MetricsGraphics.js',
                            left: 'center'
                        },
                        tooltip: {
                            trigger: 'axis',
                            axisPointer: {
                                type: 'cross',
                                animation: false,
                                label: {
                                    backgroundColor: '#ccc',
                                    borderColor: '#aaa',
                                    borderWidth: 1,
                                    shadowBlur: 0,
                                    shadowOffsetX: 0,
                                    shadowOffsetY: 0,
                                    color: '#222'
                                }
                            },
                            formatter: function (params) {
                                // console.log('!', params,data[params[2].name]);
                                if (data[params[2].name].screen.length == 0) {
                                    return
                                }
                                return (
                                    data[params[2].name].screen +
                                    '<br />' +
                                    'content: ' + params[0].value.toFixed(0) + '<br />' +
                                    'emotion: ' + params[1].value.toFixed(0) + '<br />' +
                                    'event: ' + params[2].value.toFixed(0) + '<br />' +
                                    ((params[2].value - base)).toFixed(1) +
                                    '%'
                                );
                            }
                        },
                        grid: {
                            left: '3%',
                            right: '4%',
                            bottom: '3%',
                            containLabel: true
                        },
                        xAxis: {
                            type: 'category',
                            // type: 'value',
                            data: data.map(function (item) {
                                return item.start;
                                // return item.id;
                            }),
                            axisLabel: {
                                formatter: function (value, idx) {
                                    // console.log(data[value], value);
                                    // return data[value].screen
                                }
                            },
                            boundaryGap: false
                        },
                        yAxis: {
                            axisLabel: {
                                formatter: function (val) {
                                    return (val - base) + '%';
                                }
                            },
                            axisPointer: {
                                label: {
                                    formatter: function (params) {
                                        return ((params.value - base)).toFixed(1) + '%';
                                    }
                                }
                            },
                            splitNumber: 3
                        },
                        series: [
                            {
                                name: 'L',
                                type: 'line',
                                data: data.map(function (item) {
                                    return item.emotion + base;
                                }),
                                lineStyle: {
                                    opacity: 0
                                },
                                // areaStyle: {
                                //     color: '00BFFF'
                                // },
                                stack: 'confidence-band',
                                symbol: 'none',
                                smooth: true,
                            },
                            {
                                name: 'U',
                                type: 'line',
                                data: data.map(function (item) {
                                    return item.event - item.emotion;
                                }),
                                lineStyle: {
                                    opacity: 0
                                },
                                areaStyle: {
                                    color: '#00BFFF'
                                },
                                stack: 'confidence-band',
                                symbol: 'none',
                                smooth: true,

                            },
                            {
                                type: 'line',
                                data: data.map(function (item) {
                                    return item.content + base;
                                }),
                                itemStyle: {
                                    color: '#333'
                                },
                                showSymbol: false,
                                smooth: true,
                                markArea: {
                                    itemStyle: {
                                        color: 'gray',
                                        opacity: '0.2'
                                    },
                                    data: [
                                        [
                                            {
                                                name: 'Exposition',
                                                xAxis: '11'
                                            },
                                            {
                                                xAxis: '16'
                                            }
                                        ],
                                        [
                                            {
                                                name: 'Raise',
                                                xAxis: '136'
                                            },
                                            {
                                                xAxis: '158'
                                            }
                                        ],
                                        [
                                            {
                                                name: 'Climax',
                                                xAxis: '179'
                                            },
                                            {
                                                name: 'Failing',
                                                xAxis: '288'
                                            }
                                        ],
                                        [
                                            {
                                                name: 'Ending',
                                                xAxis: '351'
                                            },
                                            {
                                                xAxis: '405'
                                            }
                                        ],
                                        [
                                            {
                                                xAxis: '508'
                                            },
                                            {
                                                xAxis: '538'
                                            }
                                        ],

                                    ],
                                    // data: data.map((d,idx, arr)=>{
                                    //     if (d?.id == 15){
                                    //         return [
                                    //             {xAxis: d.screen},
                                    //             {xAxis: arr[93]}
                                    //         ]
                                    //     }
                                    // })
                                }

                            }
                        ]
                    })
                );
            };
            function doChartTwo(data) {
                myChart.hideLoading();
                var base = -data.reduce(function (min, val) {
                    return Math.floor(Math.min(min, val.emotion));
                }, Infinity);
                myChart.setOption(
                    (option = {
                        title: {
                            text: 'rhythm_line',
                            subtext: 'Example in MetricsGraphics.js',
                            left: 'center'
                        },
                        tooltip: {
                            trigger: 'axis',
                            axisPointer: {
                                type: 'cross',
                                animation: false,
                                label: {
                                    backgroundColor: '#ccc',
                                    borderColor: '#aaa',
                                    borderWidth: 1,
                                    shadowBlur: 0,
                                    shadowOffsetX: 0,
                                    shadowOffsetY: 0,
                                    color: '#222'
                                }
                            },
                            formatter: function (params) {
                                // console.log('!', params,data[params[2].name]);
                                if (data[params[2].name].screen.length == 0) {
                                    return
                                }
                                return (
                                    data[params[2].name].screen +
                                    '<br />' +
                                    'content: ' + params[0].value.toFixed(0) + '<br />' +
                                    'emotion: ' + params[1].value.toFixed(0) + '<br />' +
                                    'event: ' + params[2].value.toFixed(0) + '<br />' +
                                    ((params[2].value - base)).toFixed(1) +
                                    '%'
                                );
                            }
                        },
                        grid: {
                            left: '3%',
                            right: '4%',
                            bottom: '3%',
                            containLabel: true
                        },
                        xAxis: {
                            type: 'category',
                            // type: 'value',
                            data: data.map(function (item) {
                                return item.start;
                                // return item.id;
                            }),
                            axisLabel: {
                                formatter: function (value, idx) {
                                    // console.log(data[value], value);
                                    // return data[value].screen
                                }
                            },
                            boundaryGap: false
                        },
                        yAxis: {
                            axisLabel: {
                                formatter: function (val) {
                                    return (val - base) + '%';
                                }
                            },
                            axisPointer: {
                                label: {
                                    formatter: function (params) {
                                        return ((params.value - base)).toFixed(1) + '%';
                                    }
                                }
                            },
                            splitNumber: 3
                        },
                        series: [
                            {
                                name: 'L',
                                type: 'line',
                                data: data.map(function (item) {
                                    return item.emotion + base;
                                }),
                                lineStyle: {
                                    opacity: 1,
                                    color: 'gray'
                                },
                                // areaStyle: {
                                //     color: '00BFFF'
                                // },
                                stack: 'confidence-band',
                                symbol: 'none',
                                smooth: true,
                            },
                            {
                                name: 'U',
                                type: 'line',
                                data: data.map(function (item) {
                                    return item.event - item.emotion;
                                }),
                                lineStyle: {
                                    opacity: 1,
                                    color: 'gray'
                                },
                                areaStyle: {
                                    color: '#00BFFF'
                                },
                                stack: 'confidence-band',
                                symbol: 'none',
                                smooth: true,

                            },
                            {
                                type: 'line',
                                data: data.map(function (item) {
                                    return item.content + base;
                                }),
                                itemStyle: {
                                    color: '#333'
                                },
                                showSymbol: false,
                                smooth: true,
                                markArea: {
                                    itemStyle: {
                                        color: 'gray',
                                        opacity: '0.2'
                                    },
                                    data: [
                                        // [
                                        //     {
                                        //         name: 'Exposition',
                                        //         xAxis: '11'
                                        //     },
                                        //     {
                                        //         xAxis: '16'
                                        //     }
                                        // ],
                                        // [
                                        //     {
                                        //         name: 'Raise',
                                        //         xAxis: '136'
                                        //     },
                                        //     {
                                        //         xAxis: '158'
                                        //     }
                                        // ],
                                        // [
                                        //     {
                                        //         name: 'Climax',
                                        //         xAxis: '179'
                                        //     },
                                        //     {
                                        //         name: 'Failing',
                                        //         xAxis: '288'
                                        //     }
                                        // ],
                                        // [
                                        //     {
                                        //         name: 'Ending',
                                        //         xAxis: '351'
                                        //     },
                                        //     {
                                        //         xAxis: '405'
                                        //     }
                                        // ],
                                        // [
                                        //     {
                                        //         xAxis: '508'
                                        //     },
                                        //     {
                                        //         xAxis: '538'
                                        //     }
                                        // ],

                                    ],
                                //     // data: data.map((d,idx, arr)=>{
                                //     //     if (d?.id == 15){
                                //     //         return [
                                //     //             {xAxis: d.screen},
                                //     //             {xAxis: arr[93]}
                                //     //         ]
                                //     //     }
                                //     // })
                                }

                            }
                        ]
                    })
                );
                myChart.on("click", params=>{
                    let index = params.dataIndex;
                    console.log(index, data[index].idx, data[index]);
                    that.showtext = data[index].screen;
                })
            };
            showList.length==0?doChart(quickLookData):doChartTwo(data);
            // doChart(data)
            option && myChart.setOption(option);
            this.showChain()
            this.mixTrend()
        },
        showChain() {
            d3.select("#chainChart").selectAll("*").remove();
            const containerWidth = d3.select('#chainChart').node().clientWidth;
            const containerHeight = d3.select('#chainChart').node().clientHeight;
            const topMargin = 10;
            const unitLen = containerHeight / 8;
            const svg = d3.select("#chainChart")
                .append('svg')
                .attr('width', '100%')
                .attr('height', '100%');
            const chain_g = svg.append('g').attr('id', 'chain_g');

            const chain_list = chain_g.selectAll("chain_list_g")
                .data(new Array(10))
                .join('g')
                .attr('class', 'chain_list_g')
                .attr('transform', (d, r, a) => `translate(0,${(unitLen) * r})`)
            chain_list.append('rect')
                .attr('class', 'person_cloumn')
                .attr('width', containerWidth)
                .attr('height', unitLen)
                .attr('x', 80)
                .attr('y', 0)
                .style('fill', (d, r) => r % 2 == 0 ? '#8da8c5' : '#ffffff')
                .style('opacity', 0.3)
            const nameList = ['Joker', 'Adam', 'Social Worker', 'Sophie', 'Clerk', 'ALFRED', 'Mom'];
            chain_g.selectAll('text')
                .data(nameList)
                .join('text')
                .attr('x', 0)
                .attr('y', (d, r) => r * unitLen + 15)
                .text(d => d)
                .style('font-size', '16px')

            const oloData = [];
            for (let i = 0; i < 15; i++) {
                let head = Math.floor(5 * Math.random());
                let len = Math.floor(5 * Math.random());
                let tmp = { head: head, len: len };
                oloData.push(tmp);
            }
            const p2p = chain_g.selectAll("olo_g")
                .data(oloData)
                .join('g')
                .attr('class', 'olo_g')
                .attr('transform', (d, r) => `translate(${(r * 80) + 100},0)`)
            p2p.append('circle')
                .attr('r', 8)
                .attr('cx', 0)
                .attr('cy', d => topMargin + d.head * (unitLen))
                .style('fill', 'black')
            const bridge = p2p.append('line')
                .attr('x1', 0)
                .attr('x2', 0)
                .attr('y1', d => topMargin + d.head * (unitLen))
                .attr('y2', d => topMargin + (d.head + d.len) * (unitLen))
                .attr('fill', 'black')
                .attr('stroke', '#C0C0C0')
                .attr('stroke-width', '5')
            p2p.append('circle')
                .attr('r', 8)
                .attr('cx', 0)
                .attr('cy', d => topMargin + (d.head + d.len) * (unitLen))
                .style('fill', 'gray')
            bridge.raise();
            this.moo()
        },
        moo() {
            const svg = d3.select("#performance_methods")
                .append('svg')
                .attr('width', '100%')
                .attr('height', '100%');

            svg.selectAll('rect')
                .data([1, 2, 3])
                .join('rect')
                .attr('width', 200)
                .attr('height', 130)
                .attr('x', 40)
                .attr('y', (d, r) => 40 + r * 190)
                .style('fill', '#8da8c5')
                .style('opacity', 0.3)

            svg.selectAll('text')
                .data([1, 2, 3])
                .join('text')
                .attr('x', 40)
                .attr('y', (d, r) => 193 + r * 190)
                .text('指导意见')
                .style('font-size', '20px')
        },

        mixTrend() {
            d3.select("#timerect").selectAll("*").remove();
            const _self = this;
            const vaildEvent = this.$store.state.event_type;
            const showList = this.$store.state.trendList;
            // console.log('事件类型', vaildEvent);
            const sluglinesMeta = this.$store.state.sluglinesMeta;
            // let initList = this.initList;
            let initList = [showList];
            // console.log(initList);
            let vaildList = []
            let timeList = []
            let mixList = []    
            const tiaoMax = 1350; 
            const leftPadding = 85;   
            // #1500px
        
            initList.forEach(element => {
                let tmp = [];
                let temp = [];
                let add = 0;
                element.forEach(slug => {
                    tmp.push(vaildEvent[slug])
                    let len = vaildEvent[slug].length
                    temp.push({ 'time': sluglinesMeta[slug]['time'], 'len': vaildEvent[slug].length, 'pos': add })
                    add = add + len;
                });
                vaildList.push(tmp)
                timeList.push(tmp)

                mixList.push({ 'context': tmp, 'time': temp })
            });
            
            let max = tiaoMax/(mixList[0].time[mixList[0].time.length-1].pos);
            mixList[0].time.forEach(e=>{
                e.len *= max;
                e.pos *= max;
            })
            //draw line and time bloak
            //#eaeaea #c0c0c0 #808080 #404040
            let timeArray = ['EARLY MORNING', 'MORNING', 'DAY', 'AFTERNOON', 'NEXT AFTERNOON',
                'LATE AFTERNOON', 'MAGIC HOUR', 'DUSK', 'EVENING', 'NIGHT', 'LATE NIGHT', 'LATER', 'SECONDS LATER', '']
            let colorTimeArray = ['#eaeaea', '#eaeaea', '#eaeaea', '#c0c0c0', '#c0c0c0',
                '#c0c0c0', '#c0c0c0', '#808080', '#808080', '#404040', '#404040', '#404040', '#404040', '#eaeaea']
            const timeColorScale = d3.scaleOrdinal().domain(timeArray).range(colorTimeArray)
            const yScale = d3.scaleLinear().domain([0, 370]).range([0, -80])
            
            let door_width = 70,
                door_height = 230,
                dot_in_door_radius = 10,
                door_padding = 90;

            const svg = d3.select('#timerect').append("svg").attr("width", "100%").attr("height", "100%")

            const trendGroup = svg.selectAll('g .trendGroup')
                .data(mixList)
                .join('g')
                .attr('class', 'trendGroup')
                // .attr('transform', (d, r) => `translate(550, ${r * door_padding + 110})`)
            trendGroup
                .selectAll('rect .timeRect')
                .data(d => d.time)
                .join('rect')
                .attr('class', 'timeRect')
                .attr('x', d => d.pos + leftPadding )
                .attr('y', 0)
                .attr('width', d => d.len)
                .attr('height', 10)
                .style('fill', d => timeColorScale(d.time))
                .style('stroke', 'white')
                .style('stroke-width', .5)
                .on('mouseover', function (event) {
                    let t = d3.select(this).data()
                    let t1 = t[0]['time']
                    let xy = [event.offsetX, event.offsetY ]
                    svg
                        .append('text')
                        .attr('id', 'context')
                        .attr('x', xy[0])
                        .attr('y', xy[1] + 10)
                        .style('font-size', '13')
                        .text(`${t1}`)

                })
                .on('mouseout', function () {
                    svg.selectAll('#context').remove()
                })
                
                return

            trendGroup
                .append('line')
                .attr('x1', 0)
                .attr('x2', 600)
                .attr('y1', 0)
                .attr('y2', 0)
                .style('stroke', 'white')
                .style('stroke-width', 1)
            const line = d3.line()
                .x((d, r) => r * 7)
                .y(d => yScale(d['sentence'].length))

            vaildList = vaildList.map(d => {
                return d.flat()
            })
            // console.log(d3.extent(vaildList.flat().map(d => d['sentence'].length)));




            const trendLine = svg.append('g').attr('class', 'trendLineG')
            const personDes = trendLine.selectAll('g .personDescribe')
                .data(vaildList)
                .join('g')
                .attr('class', 'personDescribe')
                .attr('transform', (d, r) => `translate(550, ${r * door_padding + 150})`)
            personDes
                .selectAll('rect .perDe')
                .data(d => d)
                .join('rect')
                .attr('class', 'perDe')
                .attr('x', (d, r) => r * 7)
                .attr('y', -30)
                .attr('width', d => {
                    // console.log(d);
                    return 2
                })
                .attr('height', 15)
                .style('fill', d => {
                    if (d['type'] == 'character description') {
                        return 'red'
                    }
                    else if (d['type'] == 'action interaction') {
                        return '#af9ece'
                    }
                    else {
                        return 'none'
                    }
                })




            trendLine.selectAll('path .trendLine')
                .data(vaildList)
                .join('path')
                .attr('class', 'trendLine')
                .attr('d', line.curve(d3.curveBasis))
                .attr('transform', (d, r) => `translate(550, ${r * door_padding + 150})`)
                .style('fill', 'none')
                .style('stroke', '#4e79a7')
                .style('stroke-width', 3)

            trendLine.selectAll('line .li')
                .data(vaildList)
                .join('line')
                .attr('class', 'li')
                .attr('x1', 0)
                .attr('x2', 600)
                .attr('y1', 0)
                .attr('y2', 0)
                .style('stroke', '#4e79a7')
                .style('stroke-width', 1)
                .attr('transform', (d, r) => `translate(550, ${r * door_padding + 150})`)

            const yAxis = d3.axisLeft(yScale)

            var defs = svg.append("defs");
            var filter = defs.append("filter")
                .attr("id", "drop-shadow")
                .attr("height", "130%");
            filter.append("feGaussianBlur")
                .attr("in", "SourceAlpha")
                .attr("stdDeviation", 2)
                .attr("result", "blur");
            filter.append("feOffset")
                .attr("in", "blur")
                .attr("dx", 2)
                .attr("dy", 2)
                .attr("result", "offsetBlur");

            var feMerge = filter.append("feMerge");
            feMerge.append("feMergeNode").attr("in", "offsetBlur");
            feMerge.append("feMergeNode")
                .attr("in", "SourceGraphic");

            trendLine.selectAll('line .yAxis')
                .data(vaildList)
                .join('line')
                .attr('x0', 0)
                .attr('x1', 0)
                .attr('y0', 0)
                .attr('y1', -75)
                .attr("transform", (d, r) => `translate(550, ${r * door_padding + 150})`)
                .style('fill', 'none')
                .style('stroke', '#4e79a7')
                .style('stroke-width', 1)
            // .style("filter", "url(#drop-shadow)")

            // const trendLineRect = svg.selectAll('g .lineRect')
            //         .data(vaildList)
            //             .join('g')
            //             .attr("transform", (d,r)=>`translate(550, ${(r+1)*door_padding + 100})`)
            //             .call(yAxis);










        },
    }
}
</script>

<style>
.container_rhythm {
    display: grid;
    grid-template-columns: 4fr 1fr;
    /* 第一列宽度为9，第二列宽度为1 */
    grid-gap: 10px;
    /* 列之间的间隔 */
    width: 2000px;
}

.chartContainer {
    width: 100%;
    height: 600px;
    color: #695d5d;
    /* color: #f0c313; */
}

.sub_container {
    border: solid rgb(237, 174, 174);
    /* color: ; */

}</style>