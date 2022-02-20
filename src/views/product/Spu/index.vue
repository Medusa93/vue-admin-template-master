<template>
  <div>
    <el-card style="margin: 20px 0">
      <!-- 三级联动已经是全局组件可以直接用 -->
      <CategorySelect @getCategoryId="getCategoryId" :show="!show">
      </CategorySelect>
    </el-card>
    <el-card>
      <el-button type="primary" icon="el-icon-plus" style="margin-bottom: 10px" :disabled="!category3Id" @click="scene = 1">添加SPU</el-button>
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
          <template slot-scope="{row, $index}">
            <HintButton type="success" size="mini" icon="el-icon-plus" title="添加spu"></HintButton>
            <HintButton type="warning" size="mini" icon="el-icon-edit" title="修改spu" @click="updataSpu(row)"></HintButton>
            <HintButton type="info" size="mini" icon="el-icon-info" title="查看当前spu全部sku列表"></HintButton>
            <HintButton type="danger" size="mini" icon="el-icon-delete" title="删除spu"></HintButton>
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
        :total="total">
      </el-pagination>
      </div>
      <!-- 添加spu|修改spu -->
      <SpuForm v-show="scene == 1" @changeScene="changeScene" ref="spu"></SpuForm>
      <!-- 添加sku -->
      <SkuForm v-show="scene == 2"></SkuForm>
    </el-card>
  </div>
</template>

<script>
import SpuForm from './SpuForm'
import SkuForm from './SkuForm'
export default {
  name: "Spu",
  components: {
    SpuForm,
    SkuForm
  },
  data() {
    return {      
      category3Id: '', // 三级分类id
      show: true, // 控制三级选择框是否禁用
      page: 1, // 当前页数
      limit: 3, // 当前页展示多少条数据
      records: [], // spu列表数据
      total: 0, // 数据总条数
      scene: 0, // 0:展示spu列表数据 1:添加spu|修改spu 2:添加sku
    };
  },
  methods: {
    // 三级分类自定义事件回调
    getCategoryId(params) {
      // 三级分类id
      this.category3Id = params.category3Id
      this.getSpuList()
    },
    // 获取spu列表数据
    async getSpuList(pages = 1) {
      this.page = pages
      const { page, limit, category3Id } = this
      const res = await this.$API.spu.reqSpuList(page, limit, category3Id)
      if(res.code == 200) {
        this.records = res.data.records
        this.total = res.data.total
      }
    },
    // 当分页器每页展示数据条数发生变化
    handleSizeChange(limit) {
      // 修改参数
      this.limit = limit
      // 再发请求
      this.getSpuList()
    },
    // 自定义事件回调(spuForm)
    changeScene(scene) {
      this.scene = scene
    },
    // 修改一个spu
    updataSpu(row) {
      this.scene = 1
      this.$refs.spu.initSpuData(row)
    }
  },
};
</script>
<style scoped>
</style>