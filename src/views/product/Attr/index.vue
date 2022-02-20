<template>
  <div>
    <el-card style="margin: 20px 0">
      <CategorySelect @getCategoryId="getCategoryId" :show="!isShowTable">
        </CategorySelect>
    </el-card>
    <el-card>
      <!-- 表格:展示平台属性 -->
      <div v-show="isShowTable">
        <el-button
          type="primary"
          icon="el-icon-plus"
          style="margin: 10px 0"
          :disabled="!category3Id"
          @click="addAttr"
          >添加属性</el-button
        >
        <el-table :data="attrList" border style="width: 100%">
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="150">
          </el-table-column>
          <el-table-column label="属性值列表" width="width">
            <template slot-scope="{ row, $index }">
              <el-tag
                type="success"
                v-for="(attrValue, index) in row.attrValueList"
                :key="attrValue.id"
                style="margin: 0 10px"
                >{{ attrValue.valueName }}</el-tag
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template slot-scope="{ row, $index }">
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="updateAttr(row)"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 添加属性|修改属性的结构 -->
      <div v-show="!isShowTable">
        <el-form :inline="true" ref="form" label-width="80px" :model="attrInfo">
          <el-form-item label="属性名">
            <el-input
              placeholder="请输入姓名"
              v-model="attrInfo.attrName"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!attrInfo.attrName"
          @click="addAttrValue"
          >添加属性值</el-button
        >
        <el-button type="default" @click="isShowTable = true">取消</el-button>
        <el-table
          style="width: 100%; margin: 20px 0"
          border
          :data="attrInfo.attrValueList"
        >
          <el-table-column type="index" label="序号" width="80">
          </el-table-column>
          <el-table-column prop="prop" label="属性值名称" width="width">
            <template slot-scope="{ row, $index }">
              <el-input
                placeholder="请输入属性值名称"
                size="mini"
                v-model="row.valueName"
                v-if="row.flag"
                @blur="toLook(row)"
                @keyup.native.enter="toLook(row)"
                :ref="$index"
              ></el-input>
              <span
                @click="toEdit(row, $index)"
                style="display: block"
                v-else
                >{{ row.valueName }}</span
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除${row.valueName}`"
                @onConfirm="deleteAttrValue($index)"
              >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button
         type="primary"
        size="medium" 
        @click="addOrUpdate"
        :disabled="!attrInfo.attrValueList.length"
        >
        保存
        </el-button>
        <el-button type="defalut" size="medium" @click="isShowTable = true"
          >取消</el-button
        >
      </div>
    </el-card>
  </div>
</template>

<script>
// 按需引入lodash深拷贝
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "Attr",
  data() {
    return {
      category1Id: '',
      category2Id: '',
      // 三级分类id,没有则不显示添加属性按钮
      category3Id: '',
      // 平台属性数据
      attrList: [],
      // 控制table表格显示与隐藏
      isShowTable: true,
      attrInfo: {
        attrName: "", // 属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: 0, // 三级分类id
        categoryLevel: 3, // 因为服务器需要区分是几级id
      },
    };
  },
  methods: {
    // 自定义事件回调
    getCategoryId(params) {
      this.category1Id = params.category1Id
      this.category2Id = params.category2Id
      this.category3Id = params.category3Id
      this.getAttrList();
    },
    // 获取品牌属性
    async getAttrList() {
      const { category1Id, category2Id, category3Id } = this;
      this.category3Id = category3Id;
      const res = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      );
      this.attrList = res.data;
    },
    // 添加属性值回调
    addAttrValue() {
      // attrId: 属性id 目前我们是添加属性操作还没有相应的id,目前而言带给服务器的id为undefined
      // valueName: 相应属性值的名称
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id, // 对于修改某一个属性的时候,可以在已有的属性基础上新增新的属性值(新增属性值的时候,需要把已有属性的id带上)
        valueName: "",
        flag: true,
        // 给每一个属性添加一个标记用户切换查看与编辑模式,每一个属性值可以控制自己的切换
        // 当前flag为响应式数据(数据变化视图跟着变化)
      });
      // 新添加的input框自动高亮
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus();
      });
    },
    //气泡确认框确定按钮的回调
    //最新版本的ElementUI----2.15.6，目前模板中的版本号2.13.x 所以点击确认按钮时触发事件是onConfirm不是confirm
    deleteAttrValue(index) {
      //当前删除属性值的操作是不需要发请求的
      this.attrInfo.attrValueList.splice(index, 1);
    },
    // 添加属性按钮回调
    addAttr() {
      // 切换table显示与隐藏
      this.isShowTable = false;
      // 清空数据
      // 手机三级分类id
      this.attrInfo = {
        attrName: "", // 属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: this.category3Id, // 三级分类id
        categoryLevel: 3, // 因为服务器需要区分是几级id
      };
    },
    // 修改属性按钮回调
    updateAttr(row) {
      this.isShowTable = false;
      // 数据结构当中存在对象套数组,数组里面又套对象,因此需要深拷贝解决
      // this.attrInfo = JSON.parse(JSON.stringify(row))
      this.attrInfo = cloneDeep(row);
      // 修改某一个属性时候,将相应的属性都添加上flag这个标记
      this.attrInfo.attrValueList.forEach((item) => {
        this.$set(item, "flag", false);
      });
      // console.log(row)
    },
    // 失去焦点事件,切换查看模式 显示span
    toLook(row) {
      if (!row.valueName.trim()) {
        return this.$message.error("属性值不能为空");
      }
      const isRepeat = this.attrInfo.attrValueList.some((item) => {
        if (row != item) {
          return row.valueName.trim() == item.valueName;
        }
      });
      if (isRepeat) {
        return this.$message.error("属性值不能重复");
      }
      row.flag = false;
    },
    // 切换到编辑模式
    toEdit(row, index) {
      row.flag = true;
      // 需要注意：点击span的时候，切换为input变为编辑模式，但是需要注意，对于浏览器而言，页面重绘与重拍耗时间的
      //点击span的时候，重绘重排一个input它是需要耗费事件，因此我们不可能一点击span立马获取到input
      // $nextTick渲染节点完成后调用
      this.$nextTick(() => {
        //获取相应的input表单元素实现聚焦
        this.$refs[index].focus();
      });
    },
    //保存按钮：进行添加属性或者修改属性的操作
    async addOrUpdate() {
      // 整理参数:1,如果用户添加很多属性值，且属性值为空的不应该提交给服务器
      // 提交给服务器数据当中不应该出现flag字段
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter(
        (item) => {
          // 过滤掉属性值不是空的
          if (item.valueName.trim()) {
            // 删除掉flag属性
            delete item.flag;
            return true;
          }
        }
      );
      try {
        //发请求
        await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo);
        //展示平台属性的信号量进行切换
        this.isShowTable = true
        //提示消失
        this.$message.success('保存成功')
        //保存成功以后需要再次获取平台属性进行展示
        this.getAttrList()
      } catch (err) {
        this.$message.error("保存失败");
      }
    },
  },
};
</script>
<style scoped>
</style>