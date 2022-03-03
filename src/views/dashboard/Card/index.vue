<template>
  <el-row :gutter="10">
    <el-col :span="6">
      <el-card>
        <Detail title="总销售额" count="￥126560">
          <template slot="chart">
            <span>周同比 56.67% <svg t="1646313914660" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="5749" width="20" height="20"><path d="M255.998 488.11h167.253l0.909 322.563h166.4v-322.56l167.253 0.455L506.819 165.61l-250.821 322.5z" fill="#1afa29" p-id="5750"></path></svg></span>
            <span>日同比 19.96% <svg t="1646313845755" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4864" width="20" height="20"><path d="M768.002 535.89H600.749l-0.909-322.563h-166.4v322.56l-167.253-0.455L517.181 858.39l250.821-322.5z" fill="#FD3333" p-id="4865"></path></svg></span>
          </template>
          <template slot="footer"> 日销售额￥12423 </template>
        </Detail>
      </el-card>
    </el-col>
    <el-col :span="6">
      <el-card>
        <Detail title="访问量" count="88460">
          <template slot="chart">
            <LineChart></LineChart>
          </template>
          <template slot="footer"> 日访问量1234 </template>
        </Detail>
      </el-card>
    </el-col>
    <el-col :span="6">
      <el-card>
        <Detail title="支付笔数" count="88460">
          <template slot="chart">
            <BarChart></BarChart>
          </template>
          <template slot="footer"> 转化率65% </template>
        </Detail>
      </el-card>
    </el-col>
    <el-col :span="6">
      <el-card>
        <Detail title="运营活动效果" count="78%">
          <template slot="chart">
          </template>
          <template slot="footer"> 日销售额￥12423 </template>
        </Detail>
      </el-card>
    </el-col>
  </el-row>
</template>

<script>
import Detail from "./Detail";
import LineChart from "./lineChart";
import BarChart from "./barChart";
export default {
  components: {
    Detail,
    LineChart,
    BarChart
  },
  data() {
    return {};
  },
};
</script>
<style scoped>
</style>