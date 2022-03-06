<template>
  <el-card class="box-card" style="margin: 10px 0">
    <div slot="header" class="clearfix">
      <!-- 头部左侧内容 -->
      <el-tabs v-model="activeName" class="tab">
        <el-tab-pane label="销售额" name="sale"></el-tab-pane>
        <el-tab-pane label="访问量" name="visite"></el-tab-pane>
      </el-tabs>
      <!-- 头部右侧内容 -->
      <div class="right">
        <span @click="setDay">本天</span>
        <span @click="setWeek">本周</span>
        <span @click="setMonth">本月</span>
        <span @click="setYear">本年</span>
        <el-date-picker
          v-model="date"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          size="mini"
          value-format="yyyy-MM-dd"
        >
        </el-date-picker>
      </div>
    </div>
    <div>
      <el-row :gutter="10">
        <el-col :span="18">
          <div class="charts" ref="charts"></div>
        </el-col>
        <el-col :span="6">
          <h3>门店{{ title }}排名</h3>
          <ul>
            <li>
              <span class="rindex">1</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
            <li>
              <span class="rindex">2</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
            <li>
              <span class="rindex">3</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
            <li>
              <span>4</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
            <li>
              <span>5</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
            <li>
              <span>6</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
            <li>
              <span>7</span>
              <span>肯德基</span>
              <span class="rvalue">123465</span>
            </li>
          </ul>
        </el-col>
      </el-row>
    </div>
  </el-card>
</template>

<script>
import * as echarts from "echarts";
import dayjs from "dayjs";
export default {
  data() {
    return {
      activeName: "sale",
      date: "",
      myChart: null,
    };
  },
  mounted() {
    // 初始化echarts
    this.myChart = echarts.init(this.$refs.charts);
    // 配置数据
    this.myChart.setOption({
      title: {
        text: this.title + "趋势",
      },
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
      },
      grid: {
        left: "3%",
        right: "4%",
        bottom: "3%",
        containLabel: true,
      },
      xAxis: [
        {
          type: "category",
          data: [
            "一月",
            "二月",
            "三月",
            "四月",
            "五月",
            "六月",
            "七月",
            "八月",
            "九月",
            "十月",
            "十一月",
            "十二月",
          ],
          axisTick: {
            alignWithLabel: true,
          },
        },
      ],
      yAxis: [
        {
          type: "value",
        },
      ],
      series: [
        {
          type: "bar",
          barWidth: "60%",
          data: [40, 52, 200, 334, 390, 330, 220, 110, 59, 152, 167, 268],
          color: "yellowgreen",
        },
      ],
    });
  },
  watch: {
    title() {
      this.myChart.setOption({
        title: {
          text: this.title,
        },
      });
    },
  },
  computed: {
    title() {
      return this.activeName == "sale" ? "销售额" : "访问量";
    },
  },
  methods: {
    // 本天
    setDay() {
      const day = dayjs().format("YYYY-MM-DD");
      this.date = [day, day];
    },
    // 本周
    setWeek() {
      const start = dayjs().day(1).format("YYYY-MM-DD");
      const end = dayjs().day(7).format("YYYY-MM-DD");
      this.date = [start, end];
    },
    // 本月
    setMonth() {
      const start = dayjs().startOf('month').format("YYYY-MM-DD");
      const end = dayjs().endOf('month').format("YYYY-MM-DD");
      this.date = [start, end];
    },
    // 本年
    setYear() {
      const start = dayjs().startOf('year').format("YYYY-MM-DD");
      const end = dayjs().endOf('year').format("YYYY-MM-DD");
      this.date = [start, end];
    },
  },
};
</script>
<style lang="scss" scoped>
::v-deep .el-card__header {
  border: none;
}
.clearfix {
  position: relative;
  display: flex;
  justify-content: space-between;
  .tab {
    width: 100%;
  }
}
.right {
  position: absolute;
  right: 0;
  span {
    margin-right: 10px;
  }
}
.charts {
  height: 300px;
  width: 100%;
}
ul {
  padding: 0;
  list-style: none;
  height: 300px;
  li {
    height: 14%;
    span {
      margin: 0 10px;
    }
    .rindex {
      float: left;
      width: 20px;
      height: 20px;
      line-height: 20px;
      text-align: center;
      color: #fff;
      background: #000;
      border-radius: 50%;
    }
    .rvalue {
      float: right;
    }
  }
}
</style>