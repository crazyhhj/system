<template>
    <div id="personAnalysisRounder">
        <div class="wall">
            <el-row :gutter="10">
                <el-col :xl="100">
                    <div class="grid-content head " style="background-color:#8da8c5 ;"></div>
                </el-col>
            </el-row>
            <div class="wall grid-content interface-body">
                <div class="pillar">
                    <div class="brick optional ">
                        <el-row :gutter="10" class="czjz">
                            <el-col :span="5" :offset="2">
                                <div class="sign" @click="deal_actor">Person</div>
                            </el-col>
                            <el-col :span="15">
                                <div class="person-content czjz">
                                    <el-select v-model="value" placeholder="person" class="person-select" size="medium">
                                        <el-option v-for="item in personList" :key="item.value" :label="item.label"
                                            :value="item.value">
                                        </el-option>
                                    </el-select>

                                </div>
                            </el-col>
                        </el-row>
                    </div>
                </div>
                <div class="pillar">
                    <div class="brick optional ">
                        <el-row :gutter="10" class="czjz">
                            <el-col :span="5" :offset="2">
                                <div class="sign">Chapter</div>
                            </el-col>
                            <el-col :span="15">
                                <div class="person-content">
                                    <el-slider v-model="value_screen" range show-stops :max="127">
                                    </el-slider>
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                </div>
                <div class="pillar">
                    <div class="brick optional ">
                        <el-row :gutter="10" class="czjz">
                            <el-col :span="5" :offset="2">
                                <div class="sign">Place</div>
                            </el-col>
                            <el-col :span="15">
                                <div class="person-content">
                                    <el-select v-model="value" placeholder="place" class="person-select">
                                        <el-option v-for="item in options" :key="item.value" :label="item.label"
                                            :value="item.value">
                                        </el-option>
                                    </el-select>
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                    <!-- <div class="brick optional ">
                        <el-row :gutter="10" class="czjz">
                            <el-col :span="6" :offset="6">
                                <button class="submit add czjz">add</button>
                            </el-col>
                            <el-col :span="8">
                                <button class="submit show czjz">submit-show</button></el-col>
                        </el-row>
                    </div> -->
                </div>
                <div class="pillar">
                    <div class="brick optional ">
                        <el-row :gutter="40" class="czjz">
                            <el-col :span="6" :offset="8">
                                <button class="submit add czjz">add</button>
                            </el-col>
                            <el-col :span="8">
                                <button class="submit show czjz">submit</button></el-col>
                        </el-row>
                    </div>
                </div>
            </div>
        </div>
        <div class="wall">
            <el-row :gutter="0" class="czjz">
                            <el-col :span="19" :offset="0">
                                <div class="candidate"></div>
                            </el-col>
                            <el-col :span="5" >
                            <div class="checkPerson"></div></el-col></el-row>
        </div>
        <div class="wall" style="height: 15%;">
            <div class="person-info"></div>
        </div>
        <div class="wall" style="height: 20%;">生平介绍
            <div style=" height: 90%; width: 90%;" class="sluglines">
                {{ this.lifeList[0] }}
            </div>
        </div>
        <div class="wall" style="height: 20%;">关系/社群视图</div>
        <div class="wall" style="height: 20%;">
        <redaPic/></div>
    </div>
</template>

<script>
import * as d3 from 'd3'
import redaPic from './redaPic.vue'
export default {
    components: {
    redaPic},
    data() {
        return {
            screen_num: this.$store.state.slugIndexList,
            personList: [],
            // options: [{
            //     value: '选项1',
            //     label: '黄金糕'
            // }, {
            //     value: '选项2',
            //     label: '双皮奶'
            // }, {
            //     value: '选项3',
            //     label: '蚵仔煎'
            // }, {
            //     value: '选项4',
            //     label: '龙须面'
            // }, {
            //     value: '选项5',
            //     label: '北京烤鸭'
            // }],
            options:[],
            value_screen:[12,17],
            lifeList:[],

        }
    },
    computed() {

    },
    mounted() {
        // deal_actor()
    },
    watch: {

    },
    methods: {
        deal_actor(){
            this.personList = Object.entries(this.$store.state.actorFre).sort((a,b)=>b[1]-a[1]).map((d,idx)=>{
                return {
                    value: idx,
                    label: d[0]
                }
            })
            let str = `Arthur is a struggling clown performer living in the impoverished and crime-ridden Gotham City. He faces numerous challenges and hardships, which eventually push him towards madness and a life of crime. The movie delves into Arthur's descent into darkness and his transformation into the iconic supervillain, becoming a symbol of chaos and rebellion in Gotham City. "Joker" received critical acclaim for its gripping storyline and Joaquin Phoenix's outstanding portrayal of Arthur Fleck.`;
            this.lifeList = [str];
        }
    },
}
</script>

<style>
#personAnalysisRounder {
    /* 居中 */
    margin: auto;

    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    border: solid skyblue 2px;
    padding: 1px;
    gap: 0px;
}
.head{
    height: 75px;
}

.wall {
    width: 100%;
    display: flex;
    flex-direction: column;
    border: solid rgb(159, 137, 137) 1px;
    overflow: hidden;
}

.person-info{
    /* border: solid black 3px; */
    width: 100%;
    height: 100%;
}


.pillar {
    flex: 1;

    width: 100%;
    /* border: solid blueviolet 2px; */
    display: flex;
    flex-direction: row;
    padding: 2px;
    gap: 5px;

    /* 垂直居中 */
    align-items: center;
}

.czjz {
    align-items: center;

}

.brick {
    flex: 1;
    height: 11px;
}

/* elementUI  CSS */
.el-col {
    border-radius: 4px;
}

.bg-purple-dark {
    background: #99a9bf;
}

.bg-purple {
    background: #d3dce6;
}

.bg-purple-light {
    background: #e5e9f2;
}

.grid-content {
    border-radius: 4px;
    min-height: 48px;
}

.sign {
    background: #9a9a9a;
    height: 18px;
    border-radius: 4px;
}
.candidate{
    background: #e9e9e9;
    height: 35px;
    border-radius: 9px;
    border: solid black 3px;

}
.checkPerson{
    background: #e9e9e9;
    height: 35px;
    border-radius: 9px;
    border: solid black 3px;
}
.person-content {
    background: #e9e9e9;
    height: 22px;
    border-radius: 2px;

}

.interface-body {
    padding: 7px 0 7px 0;
    background-color: #e9e9e9;
    height: 192px;
}

/* elementUI  CSS */

/* button style */
.submit {
    width: 100%;
    height: 24px;
    font-size: 15px;
    line-height: 10px;
    display: flex;

    border-radius: 3px;
}

.add {
    background: white;
    border: none;
}

.show {
    background: #3595f9;
    border: none;
}

.show:hover {
    background: rebeccapurple;
}

/* .person-select{
    
} */
.el-select {
    display: block;
    background: #fdfdfd;
    height: 11px;
    border-radius: 2px;

    /* width: 184px; */
    height: 20px;

    .el-input__inner {
        height: 25px;
    }

    .el-input__prefix,
    .el-input__suffix {
        height: 20px;
    }

    /* 下面设置右侧按钮居中 */
    .el-input__suffix {
        top: 0px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: nowrap;
        flex-direction: row;
        align-content: flex-start;
    }

    /* 输入框加上上下边是 32px + 2px =34px */
    .el-input__icon {
        line-height: 10px;
    }
}
.sluglines{
    display: inline-block;
    /* line-height: 24px;
    padding: 5px 0;
    
    margin-right: 10px; */
    
    word-wrap: break-word;
    /* word-break: break-all; */
    overflow: hidden; 
}
</style>