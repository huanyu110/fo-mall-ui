<template>
  <div class="box">
    <Row class="operation">
      <Col span="12">
        <DatePicker
          :options="options"
          @on-change="search"
          type="date"
          placement="bottom-end"
          placeholder="选择查看日期"
          style="width: 200px"
          v-model="yestDate"
        ></DatePicker>
      </Col>
    </Row>
    <Row>
      <p>
        <Alert type="success">
          这里展示历史某一天的热词数据统计，可根据需求配置每日持久化多少条数据。
        </Alert>
      </p>
    </Row>

    <div id="container"></div>
  </div>
</template>

<script>
import { Chart } from "@antv/g2";
import { getHotWordsHistory } from "@/api/index";
import affixTime from "@/components/affix-time";
import { Message } from "view-design";

export default {
  components: {
    affixTime,
  },
  data() {
    return {
      yestDate:new Date(new Date().getTime() - 24 * 60 * 60 * 1000).toString(),
      yestDate:this.$options.filters.unixToDate(
          (new Date().getTime() / 1000) - (24 * 60 * 60),
          "yyyy年MM月dd日"
        ),
      params: {
        date: this.$options.filters.unixToDate(
          (new Date().getTime() / 1000) - (24 * 60 * 60),
          "yyyy-MM-dd"
        ),
      },
      hotWordsChart: "", //图表
      hotWordsData: [], //数据
      options: {
        disabledDate: (date) => {
          if (this.endMonth) {
            let endDate = this.getDate(this.endMonth);
            return (date && date > endDate) || date > new Date();
          } else {
            return date && date > new Date();
          }
        },
      },
    };
  },
  computed: {},

  methods: {
    clickBreadcrumb(val) {
      this.params = { ...this.params, ...val };
    },
    // 初始化图表
    async search(val) {
      console.log(val)
      val ? (this.params.date = val) : "";
      const res = await getHotWordsHistory(this.params);
      if (res.success) {
        this.hotWordsData = res.result;
        this.hotWordsChart.data(this.hotWordsData);
        this.hotWordsChart.render();
        if (!this.hotWordsData) {
          Message.error("暂无数据");
        }
      }
    },
    handleClickSearch() {},
    init() {
      let chart = this.hotWordsChart;
      chart = new Chart({
        container: "container",
        autoFit: true,
        height: 500,
        padding: [50, 20, 50, 20],
      });
      chart.scale("score", {
        alias: "搜索次数",
      });

      chart.axis("keywords", {
        tickLine: {
          alignTick: false,
        },
      });
      chart.axis("score", false);

      chart.tooltip({
        showMarkers: false,
      });
      chart.interval().position("keywords*score").color("#f59b99");
      chart.interaction("element-active");
      this.hotWordsChart = chart;
      this.search();
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style lang="scss" scoped>
.affix-time {
  padding-left: 15px;

}


.box {
  min-height: 400px;
  margin-left: 50px;
}
</style>
