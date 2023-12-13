<template>
    <button @click="showRhythmLine">点击显示</button>
    <div ref="rhythmLineArea" style="height: 1000px;">

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
                renderer: 'canvas',
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
            console.log(data);
            myChart.showLoading();
            function doChart(data) {
                myChart.hideLoading();
                var base = -data.reduce(function (min, val) {
                    return Math.floor(Math.min(min, val.emotion));
                }, Infinity);
                myChart.setOption(
                    (option = {
                        title: {
                            text: 'Confidence Band',
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
                            data: data.map(function (item) {
                                return item.id;
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
                                showSymbol: false
                            }
                        ]
                    })
                );
            };
            doChart(data)
            option && myChart.setOption(option);

        },

    }
}
</script>

<style></style>