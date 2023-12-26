<template>
    <div style="width: 100%; padding-left: 50px;border: gray solid 1px;">
        <button @click="showRhythmLine">点击显示</button>
        <button @click="clear">clear</button>
        <el-slider v-model="showChapter1" :max="127" show-stops>
        </el-slider>
    </div>
    <div class="container_rhythm">
        <div class="sub_container">
            <div style="border: gray solid 1px;">
                <div class="chartContainer" ref="rhythmLineArea" id="ss"></div>
                <div class="chartContainer" id="timerect" style="height:40px;"></div>
                <div class="chartContainer" id="chainChart" style="height:200px;"></div>
            </div>
            <div class="chartContainer" style="height: 10px; padding-left: 50px; padding-right:50px; " >
                <span style="float: left; font-size: 2em;">Performance guidance {{ perGudType }}</span>
                <span style="float: right; font-size: 2em;"> dialogue with emotion</span>

            </div>
            <div class="chartContainer" id="per_any" style="height:340px; ">
                <div id="PerformanceGuidanceDetails" style="border-right: gainsboro solid 2px;">

                </div>
                <div id="emo_ring">
                </div>
            </div>
            <!-- <div class="chartContainer" id="chainChart" style="height:200px;">
                {{ showtext }}</div> -->

        </div>
        <div class="sub_container" id="performance_methods" style="width: 300px; border-left: gainsboro solid 2px;">
            <div id="performSelection" @click="guide">
                <div class="detail" id="ty" @click=""></div>
                <div class="detail" id="ff"></div>
                <div class="detail" id="bx"></div>
            </div>
            <span>emotion contrast</span>
            <div id="emo_line" style="height: 600px;">

            </div>
        </div>
    </div>
    <!-- <div id="heartLine" style="width: 100%; height: 100%;">
    </div> -->
</template>

<script>
import * as d3 from 'd3'
import * as echarts from 'echarts'
import heartLine from './heartLine.vue'
import { subwayData } from '@/chart/text.js'

export default {
    name: 'rhythm',
    components: {
        heartLine
    },
    data() {
        return {
            rhythmDetailData: this.$store.state.rhythmDetailData,
            index: 0,
            showtext: 'this is initValue',
            // showChapterList: this.$store.state.trendList,
            showChapter1: 0,
            showChapter2: 0,
            // showChapter: 0,
            screen_num: this.$store.state.slugIndexList,
            trendList:[],
            perGudType: '',


        }
    },
    mounted() {
    },
    computed: {

    },
    watch: {
        screen_num: {
            handler: function (screen_num) {

                const index = screen_num[screen_num.length - 1]

                // this.lineH(index);
            }, deep: true

        },
        showChapter1: {
            handler: function (showChapter1) {
                this.$store.commit('sendIndexToRelation', showChapter1);
                this.lineH(showChapter1);
                const init  = this.$store.state.initTrendList;
                const that = this;
                for(let i of init){
                    if(showChapter1 in i){
                        that.trendList = i
                        that.showRhythmLine();
                        console.log('be use');
                    }
                }
            }, deep: true

        }
    },
    methods: {
        clear() {
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
            // const showList = this.trendList;
            if (showList.length > 0) {
                this.showChapter1 = showList[0];
                this.showChapter2 = showList[showList.length - 1];
            }

            let data = showList.map(d => dataOrg[d]).flat()
            data.forEach((d, idx) => {
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
                myChart.on("click", params => {
                    let index = params.dataIndex;
                    console.log(index, data[index].idx, data[index]);
                    that.showtext = data[index].screen;
                })
            };
            showList.length == 0 ? doChart(quickLookData) : doChartTwo(data);
            // doChart(data)
            option && myChart.setOption(option);
            this.showChain();
            showList.length == 0 ? null : this.mixTrend();
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
                .attr('x', 20)
                .attr('y', (d, r) => r * unitLen + 15)
                .text(d => d)
                .style('font-size', '16px')

            const oloData = [];
            for (let i = 0; i < 15; i++) {
                let head = Math.floor(5 * Math.random());
                let len = Math.floor(5 * Math.random());
                let tmp = { head: head, len: len, id: i };
                oloData.push(tmp);
            }
            const p2p = chain_g.selectAll("olo_g")
                .data(oloData)
                .join('g')
                .attr('class', 'olo_g')
                .attr('transform', (d, r) => `translate(${(r * 80) + 100},0)`)
                .on('mouseover', (e, d) => {
                    d3.selectAll(`.olo_e${d.id}`).style('fill', 'red').attr('r', 12)
                })
                .on('mouseout', (e, d) => {
                    d3.selectAll(`#olo_e_${d.id}`).style('fill', 'black').attr('r', 8)
                    d3.selectAll(`#olo_e_${d.id}`).style('fill', 'gray').attr('r', 8)
                })
            p2p.append('circle')
                .attr('class', d => `olo_e${d.id}`)
                .attr('id', d => `olo_e_${d.id}`)
                .attr('r', 8)
                .attr('cx', 0)
                .attr('cy', d => topMargin + d.head * (unitLen))
                .style('fill', 'black')
            const bridge = p2p.append('line')
                .attr('class', d => `olo_e${d.id}`)
                .attr('x1', 0)
                .attr('x2', 0)
                .attr('y1', d => topMargin + d.head * (unitLen))
                .attr('y2', d => topMargin + (d.head + d.len) * (unitLen))
                .attr('fill', 'black')
                .attr('stroke', '#C0C0C0')
                .attr('stroke-width', '5')
            p2p.append('circle')
                .attr('class', d => `olo_e${d.id}`)
                .attr('id', d => `olo_e_${d.id}`)
                .attr('r', 8)
                .attr('cx', 0)
                .attr('cy', d => topMargin + (d.head + d.len) * (unitLen))
                .style('fill', 'gray')
            bridge.raise();
            this.moo()
        },
        moo() {
            d3.selectAll(".detail").selectAll("*").remove();
            const createExample = (id, tt) => {
                const svg = d3.select(id)
                    .append('svg')
                    .attr('width', '300')
                    .attr('height', '200');

                svg.append('rect')
                    .attr('width', 300)
                    .attr('height', 130)
                    .attr('x', 40)
                    .attr('y', 40)
                    .style('fill', '#8da8c5')
                    .style('opacity', 0.3)

                svg.append('text')
                    .attr('x', 40)
                    .attr('y', 193)
                    .text('指导意见')
                    .style('font-size', '20px')
                svg.selectAll('.miaoshu')
                    .data(tt)
                    .join('text')
                    .attr('class', 'miaoshu')
                    .attr('x', 40)
                    .attr('y', (d,r)=>65 + 20*r)
                    .text(d=>d)
                    .style('font-size', '20px')
            }
            let t3 = ['表现派', '它强调通过表演、行为','和身体语言来传达信息','和表达意图'];
            let t1 = [ '体验派', '斯坦尼斯拉夫斯基梳','理的表演的核心' ];
            let t2 = ['方法派', '是对“体系”的发展，','是用具体方法解','决其中的问题'];
            createExample("#ty", t1);
            createExample("#ff", t2);
            createExample("#bx", t3);
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

            let max = tiaoMax / (mixList[0].time[mixList[0].time.length - 1].pos);
            mixList[0].time.forEach(e => {
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
                .attr('x', d => d.pos + leftPadding)
                .attr('y', 0)
                .attr('width', d => d.len)
                .attr('height', 10)
                .style('fill', d => timeColorScale(d.time))
                .style('stroke', 'white')
                .style('stroke-width', .5)
                .on('mouseover', function (event) {
                    let t = d3.select(this).data()
                    let t1 = t[0]['time']
                    let xy = [event.offsetX, event.offsetY]
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

        guide(e) {
            const type = e.target.id;
            const that = this;
            this.perGudType = type
            const { Joker, ws1, ws2, ws3, young_women } = subwayData;
            const useData = this.perGudType=='ty'?Joker:this.perGudType=='ff'?ws1:ws2;
            
            const tyActionList = []

            //创建坐标系
            const width = 800, height = 400;
            const margin = {
                bottom: 20,
                top: 20,
                left: 20,
                right: 20
            }
            // const colorScale = d3.scaleBand().domain(actorList).range(["#000000","#ffffff"])
            const xScale = d3.scaleBand().domain(useData).range([margin.left, width - margin.right]);
            const yScale = d3.scaleLinear().domain([0, 101]).range([height - margin.bottom, margin.top])
            const yScale_2 = d3.scaleLinear().domain([0, 150]).range([height - margin.bottom, margin.top])
            const sizeScale = d3.scalePow().exponent(2).domain([0, d3.max(useData.map(d => d.length))]).range([5, 30])


            const xAxis = d3.axisBottom(xScale)
            const yAxis = d3.axisLeft(yScale)



            d3.select("#PerformanceGuidanceDetails").selectAll('*').remove()
            const mainSvg_org = d3.select("#PerformanceGuidanceDetails")
                .append('svg')
                .attr('width', width)
                .attr('height', height)
                .attr('transform', `translate(0,0)`)
            // const finalData = [dataRole]
            // console.log(finalData);
            const mainSvg = mainSvg_org.append("g")
            const mainSvg_x = mainSvg_org.append("g")
            const mainSvg_y = mainSvg_org.append("g")
            mainSvg_x
                .append('g')
                .attr("class", "x-lineaxis")
                .attr('transform', `translate(0,${height - margin.bottom})`)
                .call(xAxis)
                .selectAll(".x-lineaxis text")
                .attr("transform", "translate(-10,10)rotate(-45)")
                .style("text-anchor", "end")
                .style("font-size", 5)
                .style("fill", "#69a3b2")
            mainSvg_y
                .append('g')
                .attr("class", "y-lineaxis")
                .attr('transform', `translate(${margin.left},0)`)
                .call(yAxis)

            const ty_show = () => {
                mainSvg
                    .selectAll('.ring_ty')
                    .data(useData)
                    .join('circle')
                    .attr('class', 'ring_ty')
                    .attr('r', d => sizeScale(d.length))
                    .attr('cx', (d, r) => xScale(d) + 20)
                    .attr('cy', d => yScale_2(d.length))
                    .style('fill', '#8aa3c8')
                    .style('opacity', '.5')
                mainSvg
                    .selectAll('.ring_ty_2')
                    .data(useData)
                    .join('circle')
                    .attr('class', 'ring_ty_2')
                    .attr('r', d => sizeScale(d.length))
                    .attr('cx', (d, r) => xScale(d) + 20 - Math.random() * sizeScale(d.length) / 2)
                    .attr('cy', d => yScale_2(d.length) - sizeScale(d.length) * Math.random())
                    .style('fill', '#8aa3c8')
                    .style('opacity', '.5')
                mainSvg
                    .selectAll('.ring_ty_3')
                    .data(useData)
                    .join('circle')
                    .attr('class', 'ring_ty_3')
                    .attr('r', d => sizeScale(d.length) * Math.random())
                    .attr('cx', (d, r) => xScale(d) + 20 + Math.random() * sizeScale(d.length) / 2)
                    .attr('cy', d => yScale_2(d.length) - 14 * Math.random())
                    .style('fill', '#8aa3c8')
                    .style('opacity', '.5')

                const ty_line = d3.line()
                    .x((d, r) => {
                        return xScale(d) + 20
                    })
                    .y((d, r) => {
                        return yScale_2(d.length)
                    })
                    .curve(d3.curveBasis);

                mainSvg.append('g')
                    .append('path')
                    .attr('class', 'line')
                    .attr('d', ty_line(useData))
                    .attr('fill', 'none')
                    .style('stroke', d3.schemeTableau10[2])
                    .style('stroke-width', 3)
                // .attr('transform', 'translate(100,0)')
            }
            // console.log(type);
            // type == 'ty' && ty_show();
            ty_show();

            this.mixScriptLineGenerate();
        },

        lineH(index) {
            //    console.log('heartData', '??Asd',heartData);

            const emoConstrast = [
                //一般  blue 3
                'caring', 'faithful',
                'impressed',
                //坏的消极 red 7
                'guilty', 'sad',
                'ashamed', 'disgusted',
                'anxious', 'afraid',
                'disappointed',
                //坏的积极 yellow 1
                'devastated', 'furious', 'surprised', 'annoyed', 'angry',
                //好的对自己  green 1
                'joyful',
                'excited',
                'confident', 'anticipating',
                'proud',
                //好的对别人  green 2
                'hopeful', 'grateful', 'trusting', 'content',
                // 一般 gray 7
                'nostalgic', 'sentimental',
                'lonely', 'jealous',
                'apprehensive', 'prepared',
                'embarrassed',

                'nn'
            ]
            const axeConstrast = [
                -1, -1, -1,
                -3, -3, -3, -3, -3, -3, -3,
                -2, -2, -2, -2, -2,
                3, 3, 3, 3, 3,
                2, 2, 2, 2,
                1, 1, 1, 1, 1, 1, 1,
                0
            ]
            const heartData = this.$store.state.actorEmoSlug
            let actorArray = []
            actorArray = []

            for (let index = 0; index < heartData.length; index++) {
                const element = heartData[index];
                for (let tick = 0; tick < element.length; tick++) {
                    const unit = element[tick];
                    if (!actorArray[unit['name']]) {
                        actorArray[unit['name']] = []
                        actorArray[unit['name']].push([...unit['emotion']])
                    }
                    else {
                        actorArray[unit['name']].push([...unit['emotion']])
                    }

                }

            }


            let name_set = [...new Set(heartData[index].map(d => d.name))]
            let actor_emo = name_set.map(d => {
                return { 'name': d, 'emo': actorArray[d] }
            })

            const emoScale = d3.scaleOrdinal().domain(emoConstrast).range(axeConstrast)

            const line = d3.line()
                .y((d, r) => {
                    return r * 20
                })
                .x((d, r) => 100 + 40 * emoScale(d))


            d3.select("#emo_line").selectAll('*').remove()
            const svg = d3.select("#emo_line").append('svg').attr('width', '100%').attr('height', '100%').attr('transform','translate(0,30)')

            // 
            const lG = svg.append('g').attr('class', 'lg')
            let a = 0
            actor_emo.forEach(element => {
                let blank = new Array(10).fill('nn')
                let data = element['emo']
                let data_f = []
                if (element['name'] == 'JOKER') {
                    data = data.slice(index, index + 1)
                }
                data.forEach(e => {
                    data_f.push(...e);
                    data_f.push(...blank)
                })
                // lG.append('g').attr('transform', 'translate(0,'+100*a+')')
                lG.append('g').attr('transform', 'translate(0,' + ')')
                    .append('path')
                    .attr('class', 'line')
                    .attr('d', line(data_f))
                    .attr('fill', 'none')
                    .style('stroke', d3.schemeTableau10[a])
                    .style('stroke-width', 10)
                    .attr('transform', 'translate(100,0)')

                a += 1

            });




        },

        mixScriptLineGenerate() {
            // const dialogueTextData = ["Hi. Do you like to laugh? ", "It's okay. How would you know. It's okay. How would you know. It's okay. How would you know.", "I got fired. From the bank.", "Okay. Well, all the info is right there on the flyer."];
            const { Joker,ws1,ws2 } = subwayData;
            
            const dialogueTextData = this.perGudType=='ty'?Joker:this.perGudType=='ff'?ws1:ws2;
            d3.select('#emo_ring')
                .selectAll('*')
                .remove()
            let parentBox = d3.select("#emo_ring");
            let mixdia = parentBox.selectAll(".mixdia")
                .data(dialogueTextData)
                .join("div")
                .attr("class", "mixdia");
            let that = this;
            mixdia.each(function (d, idx) {
                let div = d3.select(this);
                div.append("div")
                    .attr("class", "dialogueActor")
                    .text(d);
                let emoRing = div.append("div").attr("class", "emoRing");
                that.emoRingDraw(emoRing);
                console.log(d, idx, div);
            });
        },
        emoRingDraw(div) {
            div.selectAll("svg").remove();
            let srcHeight = 70;
            let srcWidth = 70;
            const emoPendSvg = div.append("svg").attr('width', srcWidth).attr('height', srcHeight);
            let emo = new Array(3).fill(0).map(d => Math.random(d));
            let emointerval = [0, ...emo];
            let innerRadius = 7, outerRadius = 28, roundRadius = 50;
            let all = emo.reduce((a, b) => a + b);
            const srcRegion = emoPendSvg.append('g').attr("id", "srcRegion");
            const arc = d3.arc()
                .innerRadius(innerRadius)
                .outerRadius(outerRadius)
                .startAngle((d, r) => {
                    let ans = 2 * Math.PI * (emointerval.slice(0, r + 1).reduce((a, b) => a + b + .01) / all);
                    return ans;
                })
                .endAngle((d, r) => {
                    let ans = 2 * Math.PI * (emointerval.slice(0, r + 2).reduce((a, b) => a + b + .01) / all);
                    return ans;
                });
            const emoArc = srcRegion.append("g").attr("id", "emoArc");
            const emoArcs = emoArc.selectAll(".emosingle")
                .data(emo)
                .join("path")
                .attr("class", "emosingle")
                .attr("d", arc)
                .attr("fill", d => "#" + Math.floor((d * 16777215)).toString(16));
            srcRegion.attr("transform", `translate(${(srcWidth / 2)},${(srcHeight / 2)})`);
            // circle in core
            const coreCir = srcRegion.append('g').attr('id', "coreCir");
            d3.quadtree();
            coreCir.selectAll("#coreCir circle")
                .data(emo)
                .join("circle")
                .attr("cx", 0)
                .attr("cy", 0)
                .attr("r", innerRadius - 1)
                .attr("fill", d => "#" + Math.floor((d * 16777215)).toString(16))
                .style("opacity", 0.3);
        }
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
    /* width: 2000px; */
    width: 100%;
}

.chartContainer {
    /* width: 1500px; */
    width: 100%;
    height: 600px;
    color: #695d5d;

    
    /* color: #f0c313; */
}

.sub_container {
    /* border: solid rgb(237, 174, 174); */
    /* color: ; */

}

.detail {
    width: 100%;
    height: 200px;

}

.detail:hover {
    background-color: rgb(151, 178, 219);
    background-color: rgb(186, 194, 205);
}

.detail:active {
    background-color: rgb(76, 104, 147);
    /* background-color: #8aa3c8; */
}

#per_any {
    display: grid;
    grid-template-columns: 2fr 1fr;
    grid-template-rows: 349px, 349px;
    /* 第一列宽度为9，第二列宽度为1 */
    grid-gap: 10px;
}

#emo_ring {
    overflow: auto;
}
</style>