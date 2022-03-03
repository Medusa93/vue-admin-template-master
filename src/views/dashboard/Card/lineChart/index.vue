<template>
  <div class="charts" ref="charts"></div>
</template>

<script>
import * as echarts from "echarts";
export default {
  mounted() {
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(this.$refs.charts);
    var option = {
      // 隐藏x轴
      xAxis: {
        // 均分
        type: "category",
        show: false,
      },
      // 隐藏y轴
      yAxis: {
        show: false,
      },
      tooltip: {},
      // 系列
      series: [
        {
          name: '访问量',
          data: [45, 70, 20, 54, 114, 67, 95],
          type: "line",
          smooth:true, //设置折线图平滑
          //拐点的样式的设置
          itemStyle: {
            opacity: 0,
          },
          //线条的样式
          lineStyle: {
            color: "purple",
          },
          areaStyle: {
            color: {
              type: "linear",
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              colorStops: [
                {
                  offset: 0,
                  color: "purple", // 0% 处的颜色
                },
                {
                  offset: 1,
                  color: "#fff", // 100% 处的颜色
                },
              ],
              global: false, // 缺省为 false
            },
          },
        },
      ],
      //布局调试
      grid: {
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
      },
    };
    // 绘制图表
    myChart.setOption(option);
  },
};
</script>
<style scoped>
.charts {
  width: 100%;
  height: 100%;
}
</style>