<template>
    <button @click="showRhythmLine">点击显示</button>
    <div>
        <div class="chartContainer" ref="rhythmLineArea" ></div>
        <div class="chartContainer" id="chainChart" style="height:200px;"></div>
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
                                return (
                                    params[2].name +
                                    '<br />' +
                                    ((params[2].value - base) * 100).toFixed(1) +
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
                                return item.screen;
                                // return item.id;
                            }),
                            axisLabel: {
                                formatter: function (value, idx) {
                                    return idx === 0
                                        ? value
                                        : value;
                                }
                            },
                            boundaryGap: false
                        },
                        yAxis: {
                            axisLabel: {
                                formatter: function (val) {
                                    return (val - base) * 100 + '%';
                                }
                            },
                            axisPointer: {
                                label: {
                                    formatter: function (params) {
                                        return ((params.value - base) * 100).toFixed(1) + '%';
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
                                stack: 'confidence-band',
                                symbol: 'none',
                                smooth:true,
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
                                    color: 'red'
                                },
                                stack: 'confidence-band',
                                symbol: 'none',
                                smooth:true,

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
                                smooth:true,
                        
                            }
                        ]
                    })
                );
            };
            doChart(data)
            option && myChart.setOption(option);
            this.showChain()

        },
        showChain(){
            const containerWidth = d3.select('#chainChart').node().clientWidth;
            const containerHeight = d3.select('#chainChart').node().clientHeight;
            console.log(containerWidth, containerHeight);
            const svg = d3.select("#chainChart")
                            .append('svg')
                            .attr('width', '100%')
                            .attr('height', '100%');
            const chain_g = svg.append('g').attr('id', 'chain_g');

            const chain_list = chain_g.selectAll("chain_list_g")
                                    .data([1,2,3,4,5])
                                    .join('g')
                                    .attr('class','chain_list_g')
                                    .attr('transform',(d,r,a)=>`translate(0,${(containerHeight/a.length)*r})`)
            chain_list.append('rect')
                        .attr('class','person_cloumn')
                        .attr('width', containerWidth)
                        .attr('height', (d,r,a)=>containerHeight/a.length)
                        .attr('x', 0)
                        .attr('y', 0)
                        .style('fill', (d,r)=>r%2==0?'#8da8c5':'#ffffff')
            chain_list.selectAll('person_circle')
                        .data(new Array(10).fill(0))
                        .join('circle')
                        .attr('class', 'person_circle')
                        .attr('r', 8)
                        .attr('cx',(d,r)=>20+r*80)
                        .attr('cy',20)
                        // .style('fill', (d,r)=>r%2==0?'#ffffff': '#000000')
                        .style('fill', 'red')

        }
    }
}
</script>

<style>
    .chartContainer{
        width: 1500px;
        height: 400px;
        color: #ffffff;
    }
</style>