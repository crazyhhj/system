<template>
    <button @click="showRhythmLine">点击显示</button>
    <div class="container_rhythm">
        <div class="sub_container">
            <div class="chartContainer" ref="rhythmLineArea"></div>
            <div class="chartContainer" id="chainChart" style="height:200px;"></div>
        </div>
        <div class="sub_container" id="performance_methods" style="width: 300px;"></div>
    </div>
</template>

<script>
import * as d3 from 'd3'
import { mapState } from "vuex";
import * as echarts from 'echarts'

export default {
    name: 'rhythm',
    components: {},
    data() {

    },
    mounted() {
        this.moo();
    },
    computed: {

    },
    watch: {

    },
    methods: {
        showRhythmLine() {
            var dom = this.$refs.rhythmLineArea;
            // var myChart = echarts.init(chartDom);
            var myChart = echarts.init(dom, null, {
                renderer: 'svg',
                useDirtyRect: false
            });
            var option;
            const fakeData = []

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
            const data = this.$store.getters.getScheduling;
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
                                    return data[value].screen

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
                                                name:'Exposition',
                                                xAxis: '11'
                                            },
                                            {
                                                xAxis: '16'
                                            }
                                        ],
                                        [
                                            {
                                                name:'Raise',
                                                xAxis: '136'
                                            },
                                            {
                                                xAxis: '158'
                                            }
                                        ],
                                        [
                                            {
                                                name:'Climax',
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
            doChart(data)
            option && myChart.setOption(option);
            this.showChain()

        },
        showChain() {
            const containerWidth = d3.select('#chainChart').node().clientWidth;
            const containerHeight = d3.select('#chainChart').node().clientHeight;
            const topMargin = 10;
            const unitLen = containerHeight / 8;
            console.log(containerWidth, containerHeight);
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
                .attr('x',0)
                .attr('y',(d,r)=>r*unitLen + 15)
                .text(d=>d)
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
        },
        moo(){
            const svg = d3.select("#performance_methods")
                .append('svg')
                .attr('width', '100%')
                .attr('height', '100%');
            
            svg.selectAll('rect')
                .data([1,2,3])
                .join('rect')
                .attr('width', 200)
                .attr('height', 130)
                .attr('x', 40)
                .attr('y', (d,r)=>40+r*190)
                .style('fill', '#8da8c5')
                .style('opacity', 0.3)
            
            svg.selectAll('text')
                .data([1,2,3])
                .join('text')
                .attr('x', 40)
                .attr('y', (d,r)=>193+r*190)
                .text('指导意见')
                .style('font-size', '20px')
        }

    }
}
</script>

<style>
.container_rhythm{
    display: grid;
    grid-template-columns: 4fr 1fr; /* 第一列宽度为9，第二列宽度为1 */
    grid-gap: 10px; /* 列之间的间隔 */
    width: 2000px;
}
.chartContainer {
    width: 100%;
    height: 600px;
    color: #695d5d;
}
.sub_container{
    border: solid rgb(237, 174, 174);
    /* color: ; */

}
</style>