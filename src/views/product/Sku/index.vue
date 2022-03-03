<template>
  <div>
    <el-table :data="records" style="width: 100%; margin-bottom: 20px" border>
      <el-table-column type="index" label="序号" width="width">
      </el-table-column>
      <el-table-column prop="skuName" label="名称" width="width">
      </el-table-column>
      <el-table-column prop="skuDesc" label="描述" width="width">
      </el-table-column>
      <el-table-column prop="prop" label="默认图片" width="width">
        <template slot-scope="{ row, $index }">
          <img
            :src="row.skuDefaultImg"
            alt=""
            style="width: 80px; height: 80px"
          />
        </template>
      </el-table-column>
      <el-table-column prop="weight" label="重量" width="80"> </el-table-column>
      <el-table-column prop="price" label="价格" width="80"> </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{ row, $index }">
          <el-button
            type="info"
            size="mini"
            icon="el-icon-bottom"
            v-if="row.isSale == 0"
            @click="sale(row)"
          ></el-button>
          <el-button
            type="success"
            size="mini"
            icon="el-icon-top"
            v-else
            @click="cancel(row)"
          ></el-button>
          <el-button type="primary" size="mini" icon="el-icon-edit"></el-button>
          <el-button
            type="info"
            size="mini"
            icon="el-icon-info"
            @click="reqSkuById(row)"
          ></el-button>
          <el-button
            type="danger"
            size="mini"
            icon="el-icon-delete"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      align="center"
      @size-change="handleSizeChange"
      @current-change="getSkuList"
      :current-page="page"
      :page-sizes="[3, 5, 10]"
      :page-size="limit"
      layout="prev, pager, next, jumper,->,sizes, total"
      :total="total"
    >
    </el-pagination>
    <el-drawer title="我是标题" :visible.sync="show" direction="rtl" size="50%">
      <el-row>
        <el-col :span="5">名称</el-col>
        <el-col :span="16">{{ skuInfo.skuName }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">描述</el-col>
        <el-col :span="16">{{ skuInfo.skuDesc }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">价格</el-col>
        <el-col :span="16">{{ skuInfo.price }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">平台属性</el-col>
        <el-col :span="16">
          <el-tag
            type="success"
            v-for="attr in skuInfo.skuAttrValueList"
            :key="attr.id"
            >{{ attr.attrName }}</el-tag
          >
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="5">商品图片</el-col>
        <el-col :span="16">
          <el-carousel height="400px" style="width: 400px">
            <el-carousel-item
              v-for="item in skuInfo.skuImageList"
              :key="item.id"
            >
              <img :src="item.imgUrl" alt="" style="width: 400px" />
            </el-carousel-item>
          </el-carousel>
        </el-col>
      </el-row>
    </el-drawer>
  </div>
</template>

<script>
export default {
  name: "Sku",
  data() {
    return {
      page: 1, // 当前页码
      limit: 10, // 每页显示数据条数
      records: [], // sku数据列表
      total: 0, // sku数据总条数
      skuInfo: {}, //sku对象数据
      show: false,
    };
  },
  mounted() {
    this.getSkuList();
  },
  methods: {
    //获取sku列表数据的方法
    async getSkuList(pages = 1) {
      this.page = pages;
      const { page, limit } = this;
      const res = await this.$API.sku.reqSkuList(page, limit);
      if (res.code == 200) {
        this.records = res.data.records;
        this.total = res.data.total;
      }
    },
    handleSizeChange(limit) {
      this.limit = limit;
      this.getSkuList();
    },
    // 上架
    async sale(row) {
      const res = await this.$API.sku.reqSale(row.id);
      if (res.code == 200) {
        row.isSale = 1;
        this.$message.success("上架成功");
      }
    },
    // 下架
    async cancel(row) {
      const res = await this.$API.sku.reqCancel(row.id);
      if (res.code == 200) {
        row.isSale = 0;
        this.$message.success("下架成功");
      }
    },
    //获取SKU详情的
    async reqSkuById(sku) {
      this.show = true;
      const res = await this.$API.sku.reqSkuById(sku.id);
      if (res.code == 200) {
        console.log(res);
        this.skuInfo = res.data;
      }
    },
  },
};
</script>
<style>
.el-carousel {
  border: 2px solid #eee;
}
.el-carousel__item h3 {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
}
.el-carousel__button {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: red;
}
</style>
<style lang="scss" scoped>
.el-col {
  margin: 10px 20px;
}
.el-col-5 {
  text-align: right;
}
.el-tag {
  margin-right: 10px;
}
</style>