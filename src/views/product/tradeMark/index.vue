<template>
  <div>
    <!-- 按钮 -->
    <el-button type="primary" icon="el-icon-plus" @click="showDialog"
      >添加</el-button
    >
    <!-- 表格组件 -->
    <el-table :data="list" style="width: 100%">
      <el-table-column type="index" label="序号" width="80px" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称"> </el-table-column>
      <el-table-column prop="prop" label="品牌logo">
        <template slot-scope="{ row, $index }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 100px" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作">
        <template slot-scope="{ row, $index }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="updateTradeMark(row)"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteTradeMark(row)">删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      style="margin-top: 20px; text-align: center"
      @size-change="handleSizeChange"
      @current-change="getPageList"
      :current-page="page"
      :page-sizes="[3, 5, 10]"
      :page-size="limit"
      :pager-count="7"
      layout="prev, pager, next, jumper, -> ,sizes ,total"
      :total="total"
    >
    </el-pagination>
    <!-- 对话框 -->
    <el-dialog
      :title="tmform.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <el-form ref="tmform" :model="tmform" :rules="rules" style="width: 80%">
        <el-form-item label="品牌名称" label-width="100px" prop="tmName">
          <el-input autocomplete="off" v-model="tmform.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" label-width="100px" prop="logoUrl">
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmform.logoUrl" :src="tmform.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "tradeMark",
  data() {
    // 自定义规则
    var validatePass = (rule, value, callback) => {
      if(value.length < 2 || value.length > 10) {
        callback(new Error('品牌名称2-10位'))
      }else {
        callback()
      }
    }
    return {
      // 分页器当前页码
      page: 1,
      // 当前页数展示数据条数
      limit: 3,
      // 总数据条数
      total: 0,
      // 品牌数据列表
      list: [],
      // 控制对话框显示与隐藏
      dialogFormVisible: false,
      // 表单信息
      tmform: {
        tmName: "",
        logoUrl: "",
      },
      // 表单校验规则
      rules: {
        // 品牌名称校验 自定义校验规则
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          {
            validator: validatePass,
            trigger: "change",
          },
        ],
        // 品牌logo校验规则
        logoUrl: [{ required: true, message: "请上传logo图片" }],
      },
    };
  },
  methods: {
    // 获取品牌列表数据
    async getPageList(pager = 1) {
      this.page = pager;
      const { page, limit } = this;
      const { data: res } = await this.$API.tradeMark.reqTradeMarkList(
        page,
        limit
      );
      this.total = res.total;
      this.list = res.records;
    },
    // 改变当前页数显示数据条数
    handleSizeChange(limit) {
      this.limit = limit;
      this.getPageList();
    },
    // 点击显示添加品牌对话框
    showDialog() {
      // 显示对话框
      this.dialogFormVisible = true;
      // 清除数据
      this.tmform = { tmName: "", logoUrl: "" };
    },
    // 修改品牌
    updateTradeMark(row) {
      console.log(row);
      // row: 当前品牌信息
      // 显示对话框
      this.dialogFormVisible = true;
      // 将已有品牌信息赋值给tmform进行展示
      // 浅拷贝防止同步更改tmform信息
      this.tmform = { ...row };
    },
    // 图片上传成功
    handleAvatarSuccess(res, file) {
      // res: 上传成功后返回前端数据
      // file: 上传成功后服务器返回前端的数据
      this.tmform.logoUrl = res.data;
    },
    // 图片上传之前
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    // 添加按钮 (添加品牌|修改品牌)
    addOrUpdateTradeMark() {
      this.$refs.tmform.validate(async (success) => {
        if (success) {
          this.dialogFormVisible = false;
          // 发请求 (添加品牌|修改品牌)
          const res = await this.$API.tradeMark.reqAddOrUpadteTradeMark(
            this.tmform
          );
          if (res.code == 200) {
            this.$message.success(
              this.tmform.id ? "修改品牌成功" : "添加品牌成功"
            );
            // 添加或修改后需要重新加载数据
            // 添加品牌停留在第一页,修改品牌停留在当前页
            this.getPageList(this.tmform.id ? this.page : 1);
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    // 删除品牌操作
    deleteTradeMark(row) {
        this.$confirm(`你确定删除${row.tmName}, 是否继续?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
          // 用户点击确定按钮时
          //向服务器发起请求
          const res = await this.$API.tradeMark.reqDeleteTradeMark(row.id)
          console.log(res)
          if(res.code == 200) {
            this.$message({
              type: 'success',
              message: '删除成功!'
            });  
            //再次获取品牌列表数据
            this.getPageList(this.list.length > 1? this.page : this.page-1)          
          }
        }).catch(() => {
          // 用户点击取消按钮时
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });      
    }
  },
  mounted() {
    this.getPageList();
  },
};
</script>
<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>