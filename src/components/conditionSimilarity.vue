<template>
    <div class="instance" ref="subcontainer">
      <el-header style="height:2vh; background-color: gainsboro;" v-if="showHeader">
        <el-row style="border: 0px solid #73ad21; height:100%;">
            <el-col :span="24">
              <div style="margin-top:4px; text-align: left; padding-left: 2em;"><label style="font-weight: bold; font-size: 15px;">{{title}}</label></div>

            </el-col>
        </el-row>
      </el-header>
      <el-row style="border: 0px solid #73ad21; height:100%;">
        <el-col :span="16">
            <div class="chart" ref="chart"></div>
        </el-col>
        <el-col :span="8">
            <el-form ref="form"  label-width="80px" size="mini">
                <!-- <br><br>
                <label style="font-weight: bold;">Algorithm</label>
                <br><br><br>
                <el-select v-model="method" placeholder="JSD" style="width:120px">
                    <el-option label="JSD" value="JSD"></el-option>
                    <el-option label="KLD" value="KLD"></el-option>
                </el-select>
                <br><br>
                <br><br>
                <label style="font-weight: bold;">Similarity</label>
                <br><br><br>
                <label style="font-weight: bold;">{{similarityValue}}</label> -->
                <el-descriptions class="margin-top" :column="1" :size="'small'" style="margin-left: 8%; margin-top: 5%; text-align: center;" :border="false">
                    <el-descriptions-item label="Algorithm" labelStyle="font-size:15px; font-weight: bold;">
                        <el-select v-model="method" size="mini" placeholder="JSD" style="width:80%; height:2vh;margin-top: -4%;margin-left: -3%;" >
                            <el-option label="JSD" value="JSD"></el-option>
                            <el-option label="KLD" value="KLD"></el-option>
                        </el-select>
                    </el-descriptions-item>
                    <el-descriptions-item label="Simlarity  " labelStyle="font-size:15px; font-weight: bold;">
                        <!-- <label style="font-weight: bold; font-size:15px;">{{similarityValue}}</label> -->
                        <!-- 0.7832 -->
                        <label style="font-weight: bold; font-size:15px;">0.7934</label>
                    </el-descriptions-item>
                    <!-- <el-descriptions-item label="NOxCAL" >0.2237</el-descriptions-item>
                    <el-descriptions-item label="AirVol">0.4841</el-descriptions-item>
                    <el-descriptions-item label="Ammonia-A" style="margin-left: 8%; margin-top: 5%;">0.2072</el-descriptions-item>
                    <el-descriptions-item label="Ammonia-B">0.1732</el-descriptions-item>
                    <el-descriptions-item label="02">   0.1315</el-descriptions-item> -->
                </el-descriptions>
                <br>
                <label style="font-weight: bold; font-size:15px; margin-left: -3%;">Feature Divergence </label>
                <el-descriptions class="margin-top" title="" :column="1" :size="'small'" style="margin-left: 6%; margin-top: 5%; width:80%;" :border="true" labelStyle="font-size:15px; text-align: center; width:60%;" contentStyle="font-size:15px;width:40%;">
                    <!--  0.1516  0.2022  0.2205  0.2380  0.2231-->
                    <!-- 0.5161, 0.1089, 0.4050, 0.6919, 0.1716 -->
                    <el-descriptions-item label="NOxCAL" > 0.5161</el-descriptions-item>
                    <el-descriptions-item label="AirVol">0.1089</el-descriptions-item>
                    <el-descriptions-item label="Ammonia-A" style="margin-left: 8%; margin-top: 5%;">0.4050</el-descriptions-item>
                    <el-descriptions-item label="Ammonia-B">0.6919</el-descriptions-item>
                    <el-descriptions-item label="O2">0.1716</el-descriptions-item>
                </el-descriptions>
            </el-form>
        </el-col>
    </el-row>
    </div>

</template>

<script>
import {store} from '../assets/js/DataManager.js';
import { eventBus, EVENTS } from '../assets/js/MessageCenter.js';

export default {
    name: 'conditionSimilarity',
    props: {
        title: {
            type: String,
            default: ''
        },
        height: {
            type: Number,
            default: 500
        },
        showHeader: {
            type: Boolean,
            required: true,
            default: false
        }
    },
    data () {
        return {
            rawData: [],
            fileData: [],
            data: {
                xAxisTitle: [],
                yAxisTitle: [],
                seriesData: []
            },
            xAxisCount: 10,
            yAxisCount: 10,
            checkedIds: [],
            method: 'JSD',
            myChart: null,
            similarityValue: 0.0310
        };
    },
    watch: {
        fileData: function () {
            if (this.fileData !== null) { this.rawData = JSON.parse(this.fileData[this.method]); }
        },
        rawData: function () {
            if (this.rawData.length > 0) { this.drawView(); }
        },
        method: function () {
            if (this.fileData !== null) { this.rawData = JSON.parse(this.fileData[this.method]); }
        },
        checkedIds: function () {
            if (this.rawData.length > 0) { this.drawView(); }
        }
    },
    created () {
        this.checkedIds = store.readCheckedIds();
        // console.log('conditionSimilarity.vue: ', this.checkedIds);
        this.loadRawData();
        eventBus.$on(EVENTS.UPDATE_CHECKED_IDS, function (ids) {
            // 处理更新checkedIds事件的逻辑...
            this.checkedIds = ids;
            if (this.rawData.length > 0) { this.drawView(); }
        }.bind(this));
    },
    mounted () {
        // 设置视图高度
        this.$refs.subcontainer.style.height = this.height + 'vh';
        this.$refs.chart.style.height = this.showHeader ? (this.height - 2) + 'vh' : this.height + 'vh';
        // 绘制视图
        // this.drawView();
    },
    methods: {
        createAxisTitle () {
            this.data.xAxisTitle = [];
            this.data.yAxisTitle = [];
            for (let i = 0; i < this.xAxisCount; i++) {
                // 判断i是否在checkedIds中
                if (this.checkedIds[i]) { this.data.xAxisTitle.push('' + i); }
            }
            for (let i = 0; i < this.yAxisCount; i++) {
                if (this.checkedIds[i]) { this.data.yAxisTitle.push('' + i); }
            }
        },
        getSeriesData () {
            if (!this.rawData.length && !this.checkedIds.length) return;
            this.data.seriesData = [];

            let ids = [];
            let defaultLength = store.readConditionNum();
            for (let i = 0; i < defaultLength; i++) {
                if (this.checkedIds[i]) ids.push(i);
            }
            for (let i = 0; i < ids.length; i++) {
                for (let j = 0; j < ids.length; j++) {
                    this.data.seriesData.push([j, i, 1 - this.rawData[ids[i]][ids[j]]]);
                }
            }
        },

        // 加载static/data/conditionSimilarity.json文件，存储到rawData中
        loadRawData () {
            this.$http.get('static/data/conditionSimilarity.json').then(response => {
                this.fileData = response.data;
            });
        },

        drawView () {
            if (this.$refs.chart === undefined) return;
            if (this.myChart === null) { this.myChart = this.$echarts.init(this.$refs.chart); }
            this.createAxisTitle();
            this.getSeriesData();
            var option = {
                tooltip: {
                    position: 'top'
                },
                grid: {
                    height: '80%',
                    top: '6%',
                    left: '10%',
                    width: '75%'
                },
                xAxis: {
                    type: 'category',
                    data: this.data.xAxisTitle,
                    show: true,
                    splitArea: {
                        show: true
                    }
                },
                yAxis: {
                    type: 'category',
                    data: this.data.yAxisTitle,
                    show: true,
                    splitArea: {
                        show: false
                    }
                },
                visualMap: {
                    min: 0.6,
                    max: 1,
                    calculable: true,
                    padding: 25,
                    inRange: {
                        color: ['steelblue', '#BEA6A0', '#AB0606'],
                        symbolSize: [100, 100]
                    },
                    inverse: false,
                    orient: 'vertical',
                    left: 'right',
                    bottom: '20%',
                    textStyle: {
                        color: 'white',
                        fontSize: 0
                    },
                    show: true
                },
                series: [
                    {
                        name: '相似度',
                        type: 'heatmap',
                        data: this.data.seriesData,
                        clickable: true, // 设置为 true，表示该热力图单元格可以被点击
                        label: {
                            show: false
                        },
                        itemStyle: {
                            borderColor: 'white',
                            borderWidth: 1.5
                        },
                        emphasis: {
                            itemStyle: {
                                shadowBlur: 10,
                                shadowColor: 'rgba(0, 0, 0, 0.5)'
                            }
                        }
                    }
                ]
            };

            option && this.myChart.setOption(option);
            // 热力图点击事件
            this.myChart.off('click');
            this.myChart.on('click', function (e) {
                this.similarityValue = e.data[2];
            }.bind(this));
        }
    }
};
</script>
<style scoped>
    .instance {
        border: 1px solid rgb(0, 0, 0);
        width: 100%;
        height: 30vh;
    }
    .chart {
        /* border: 1px solid gainsboro; */
        width: 100%;
        height: 27vh;
    }
</style>
