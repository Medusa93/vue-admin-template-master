<template>
  <div>
    <el-form :inline="true" class="demo-form-inline" :model="cForm">
      <el-form-item label="一级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category1Id"
          @change="handler1"
          :disabled="show"
        >
          <el-option
            :label="c1.name"
            :value="c1.id"
            v-for="(c1, index) in list1"
            :key="c1.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category2Id"
          @change="handler2"
          :disabled="show"
        >
          <el-option
            :label="c2.name"
            :value="c2.id"
            v-for="(c2, index) in list2"
            :key="c2.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category3Id"
          @change="handler3"
          :disabled="show"
        >
          <el-option
            :label="c3.name"
            :value="c3.id"
            v-for="(c3, index) in list3"
            :key="c3.id"
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "CategorySelect",
  props: ['show'],
  data() {
    return {
      // 一级分类数据列表
      list1: [],
      // 二级分类数据列表
      list2: [],
      // 三级分类数据列表
      list3: [],
      // 表单数据
      cForm: {
        category1Id: "",
        category2Id: "",
        category3Id: "",
      },
    };
  },
  mounted() {
    this.getCategory1();
  },
  methods: {
    // 获取一级分类数据
    async getCategory1() {
      const res = await this.$API.attr.reqCategory1List();
      if (res.code == 200) {
        this.list1 = res.data;
      }
    },
    // 一级分类的select事件回调
    async handler1(id) {
      // 清除数据
      this.list2 = []
      this.list3 = []
      this.cForm.category2Id = ''
      this.cForm.category3Id = ''
      // 解构一级分类id
      const { category1Id } = this.cForm;
      // 获取二级分类数据
      const res = await this.$API.attr.reqCategory2List(category1Id);
      if (res.code == 200) {
        this.list2 = res.data;
      }
    },
    // 二级分类的select事件回调
    async handler2(id) {
      // 清除数据
      this.list3 = []
      this.cForm.category3Id = ''
      // 解构二级分类id
      const { category2Id } = this.cForm;
      // 获取三级分类数据
      const res = await this.$API.attr.reqCategory3List(category2Id);
      if (res.code == 200) {
        this.list3 = res.data;
      }
    },
    // 三级分类的select事件回调
    handler3() {
      // 解构三级分类id
      const { category1Id, category2Id, category3Id } = this.cForm
      this.$emit('getCategoryId', {category1Id, category2Id, category3Id})
    }
  },
};
</script>
<style scoped>
</style>