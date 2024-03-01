<template>
  <div>
    <div v-if="this.admin.loginType == 'admin'">
      <div style="margin: 20px 0">
        <el-select
          class="input"
          v-model="timeRange"
          placeholder="请选择"
          @change="load"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </div>
      <el-card>
        <div id="line" style="width: 100%; height: 400px"></div>
      </el-card>
    </div>
    <div v-else>
      <div style="margin: 20px 0; padding-top: 100px">
        <el-carousel :interval="4000" type="card" height="400px">
          <el-carousel-item v-for="item in showData" :key="item">
            <div style="width: 100%; height: 100%; position: relative">
              <el-image
                :src="item.cover"
                style="height: 100%; width: 100%"
              ></el-image>
              <h2
                style="
                  position: absolute;
                  top: 50%;
                  left: 50%;
                  transform: translate(-50%, -50%);
                  color: aqua;
                "
              >
                {{ item.name }}
              </h2>
              <h4
                style="
                  position: absolute;
                  top: 60%;
                  left: 50%;
                  transform: translate(-50%, -50%);
                "
              >{{ item.author }}</h4>
            </div>
          </el-carousel-item>
        </el-carousel>
      </div>
    </div>
  </div>
</template>

<script>
import Cookies from "js-cookie";
import request from "@/utils/request";
import * as echarts from "echarts";

const option = {
  title: {
    text: "图书借还统计",
  },
  tooltip: {
    trigger: "axis",
  },
  legend: {
    data: ["借书数量", "还书数量"],
  },
  grid: {
    left: "3%",
    right: "4%",
    bottom: "3%",
    containLabel: true,
  },
  toolbox: {
    feature: {
      saveAsImage: {},
    },
  },
  xAxis: {
    type: "category",
    boundaryGap: false,
    data: [],
  },
  yAxis: {
    type: "value",
  },
  series: [
    {
      name: "借书数量",
      type: "line",
      stack: "Total",
      smooth: true,
      data: [],
    },
    {
      name: "还书数量",
      type: "line",
      stack: "Total",
      smooth: true,
      data: [],
    },
  ],
};

export default {
  data() {
    return {
      admin: Cookies.get("admin") ? JSON.parse(Cookies.get("admin")) : {},
      lineBox: null,
      recommendData: [],
      randomValues: [],
      showData:[],
      timeRange: "week",
      options: [
        { label: "最近一周", value: "week" },
        { label: "最近一个月", value: "month" },
        { label: "最近两个月", value: "month2" },
        { label: "最近三个月", value: "month3" },
      ],
    };
  },
  mounted() {
    if (this.admin.loginType == "admin") {
      this.load();
    } else {
      this.loadRecommend();
    }
  },
  methods: {
    load() {
      if (!this.lineBox) {
        this.lineBox = echarts.init(document.getElementById("line"));
      }
      request.get("/borrow/lineCharts/" + this.timeRange).then((res) => {
        option.xAxis.data = res.data.date;
        option.series[0].data = res.data.borrow;
        option.series[1].data = res.data.retur;
        this.lineBox.setOption(option);
      });
    },
    loadRecommend() {
      let params = {
        isRecommend: 1,
      };
      request
        .get("/book/page", {
          params: params,
        })
        .then((res) => {
          if (res.code === "200") {
            this.recommendData = res.data.list;
            console.log(this.recommendData);
            if (this.recommendData.length == 0) {
              // 要获取的部分值的数量
              const count = 3;
              request.get("/book/list").then((res) => {
                console.log("是", res.data);
                this.recommendData = res.data;
                // 从数组中随机获取部分值
                for (let i = 0; i < count; i++) {
                  // 生成一个随机索引
                  const randomIndex = Math.floor(
                    Math.random() * this.recommendData.length
                  );

                  // 将对应索引的元素添加到结果数组中
                  this.randomValues.push(this.recommendData[randomIndex]);

                  // 从原数组中移除已经获取的值，避免重复选择
                  this.recommendData.splice(randomIndex, 1);
                }
              });
 this.showData = this.randomValues
            }else{
              this.showData = this.recommendData;
            }
            console.log("随机",);
          }
        });
    },
  },
};
</script>

<style>
.input {
  width: 300px;
}
.el-carousel__item h3 {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 200px;
  margin: 0;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
}
</style>
