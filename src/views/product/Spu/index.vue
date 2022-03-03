<template>
  <div>
    <el-card style="margin: 20px 0">
      <!-- 三级联动已经是全局组件可以直接用 -->
      <CategorySelect @getCategoryId="getCategoryId" :show="scene != 0">
      </CategorySelect>
    </el-card>
    <el-card>
      <el-button
        type="primary"
        icon="el-icon-plus"
        style="margin-bottom: 10px"
        :disabled="!category3Id"
        @click="addSpu"
        >添加SPU</el-button
      >
      <!-- 底部是三部分进行切换 -->
      <div v-show="scene == 0">
        <!-- spu列表展示 -->
        <el-table :data="records" style="width: 100%" border>
          <el-table-column type="index" label="序号" width="80">
          </el-table-column>
          <el-table-column label="spu名称" prop="spuName" width="width">
          </el-table-column>
          <el-table-column label="spu描述" prop="description" width="width">
          </el-table-column>
          <el-table-column label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <HintButton
                type="success"
                size="mini"
                icon="el-icon-plus"
                title="添加sku"
                @click="addSku(row)"
              ></HintButton>
              <HintButton
                type="warning"
                size="mini"
                icon="el-icon-edit"
                title="修改spu"
                @click="updataSpu(row)"
              ></HintButton>
              <HintButton
                type="info"
                size="mini"
                icon="el-icon-info"
                title="查看当前spu全部sku列表"
                @click="handler(row)"
              ></HintButton>
              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <HintButton
                  type="danger"
                  size="mini"
                  icon="el-icon-delete"
                  title="删除spu"
                  slot="reference"
                ></HintButton>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          style="text-align: center"
          @size-change="handleSizeChange"
          @current-change="getSpuList"
          :current-page="page"
          :page-sizes="[3, 5, 10]"
          :page-size="limit"
          layout="prev, pager, next, jumper, ->, sizes, total"
          :total="total"
        >
        </el-pagination>
      </div>
      <!-- 添加spu|修改spu -->
      <SpuForm
        v-show="scene == 1"
        @changeScene="changeScene"
        ref="spu"
      ></SpuForm>
      <!-- 添加sku -->
      <SkuForm
        v-show="scene == 2"
        ref="sku"
        @changeScenes="changeScenes"
      ></SkuForm>
      <el-dialog
        :title="`${spu.spuName}的sku列表`"
        :visible.sync="dialogTableVisible"
        @close="close"
      >
        <el-table :data="skuList" style="width: 100%" v-loading="loading">
          <el-table-column prop="skuName" label="名称" width="width">
          </el-table-column>
          <el-table-column prop="price" label="价格" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="weight" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="图片" width="width">
            <template slot-scope="{row, $index}">
              <img :src="row.skuDefaultImg" alt="" style="width:100px; height:100px">
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import SpuForm from "./SpuForm";
import SkuForm from "./SkuForm";
export default {
  name: "Spu",
  components: {
    SpuForm,
    SkuForm,
  },
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "", // 三级分类id
      page: 1, // 当前页数
      limit: 3, // 当前页展示多少条数据
      records: [], // spu列表数据
      total: 0, // 数据总条数
      scene: 0, // 0:展示spu列表数据 1:添加spu|修改spu 2:添加sku
      spu: {},
      dialogTableVisible: false,
      skuList: [],
      loading: true,
    };
  },
  methods: {
    // 三级分类自定义事件回调
    getCategoryId(params) {
      this.category1Id = params.category1Id;
      this.category2Id = params.category2Id;
      // 三级分类id
      this.category3Id = params.category3Id;
      this.getSpuList();
    },
    // 获取spu列表数据
    async getSpuList(pages = 1) {
      this.page = pages;
      const { page, limit, category3Id } = this;
      const res = await this.$API.spu.reqSpuList(page, limit, category3Id);
      if (res.code == 200) {
        this.records = res.data.records;
        this.total = res.data.total;
      }
    },
    // 当分页器每页展示数据条数发生变化
    handleSizeChange(limit) {
      // 修改参数
      this.limit = limit;
      // 再发请求
      this.getSpuList();
    },
    // 自定义事件回调(spuForm)
    changeScene({ scene, flag }) {
      // flag是用来判断保存按钮是修改还是添加
      this.scene = scene;
      if (flag == "修改") {
        this.getSpuList(this.page);
      } else {
        this.getSpuList();
      }
    },
    // 自定义事件回调(skuForm)
    changeScenes() {
      this.scene = 0;
    },
    // 修改一个spu
    updataSpu(row) {
      this.scene = 1;
      this.$refs.spu.initSpuData(row);
    },
    // 添加SPU
    addSpu() {
      this.scene = 1;
      this.$refs.spu.addSpuData(this.category3Id);
    },
    // 删除spu
    async deleteSpu(row) {
      const res = await this.$API.spu.reqDeleteSpu(row.id);
      if (res.code == 200) {
        this.$message.success("删除成功");
        //代表SPU个数大于1删除的时候停留在当前页，如果SPU个数小于1 回到上一页
        this.records.length > 1
          ? this.getSpuList(this.page)
          : this.getSpuList(this.page - 1);
      }
    },
    // 添加sku
    addSku(row) {
      // 切换场景2
      this.scene = 2;
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
    },
    // 查看sku列表数据
    async handler(spu) {
      this.dialogTableVisible = true
      //保存spu信息
      this.spu = spu;
      //获取sku列表的数据进行展示      
      const res = await this.$API.spu.reqSkuList(spu.id);
      if(res.code == 200) {
        this.loading = false
        this.skuList = res.data
      }
    },
    // 关闭对话框回调
    close() {
      this.loading = true
      // 清除sku列表数据 
      this.skuList = []
    }
  },
};
</script>
<style scoped>
</style>