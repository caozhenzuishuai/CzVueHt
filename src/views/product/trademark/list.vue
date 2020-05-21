<template>
  <div>
    <el-button
      type="primary"
      icon="el-icon-plus"
      @click="showAdd"
      style="margin-bottom:20px"
      >添加</el-button
    >

    <el-table v-loading="loading" :data="trademarks" border stripe>
      <el-table-column label="序号" type="index" width="80" align="center">
      </el-table-column>

      <el-table-column prop="tmName" label="品牌名称"> </el-table-column>
      <el-table-column label="姓名">
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 60px;" />
        </template>
      </el-table-column>
      <el-table-column prop="address" label="操作">
        <template slot-scope="{ row, $index }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="showUpdate(row)"
            >修改</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteTrademark(row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      background
      style="text-align: center; margin-top: 20px;"
      :current-page="page"
      :page-sizes="[3, 6, 9, 12]"
      :page-size="limit"
      layout="prev, pager, next, jumper, ->, sizes, total"
      :total="total"
      @current-change="getTrademarks"
      @size-change="handleSizeChange"
    >
    </el-pagination>
    <el-dialog :title="form.id ? '更新' : '添加'" :visible.sync="isShowDialog">
      <el-form :model="form" style="width: 80%" :rules="rules" ref="tmForm">
        <el-form-item
          label="品牌名称"
          :label-width="formLabelWidth"
          prop="tmName"
        >
          <el-input
            v-model="form.tmName"
            autocomplete="off"
            placeholder="请输入品牌名称"
          ></el-input>
        </el-form-item>
        <el-form-item
          label="品牌LOGO"
          :label-width="formLabelWidth"
          prop="logoUrl"
        >
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleLogoSuccess"
            :before-upload="beforeLogoUpload"
          >
            <img v-if="form.logoUrl" :src="form.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="isShowDialog = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTrademark"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "TrademarkList",
  data() {
    return {
      loading: false,
      trademarks: [],
      total: 0,
      page: 1,
      limit: 3,
      isShowDialog: false,
      form: {
        tmName: "",
        logoUrl: ""
      },
      formLabelWidth: "100px",
      rules: {
        /*
          品牌名称:
              必须输入   输入过程中触发校验
              长度必须在2-10个之间   失去焦点时触发校验
          品牌LOGO:
              必须有
          */
        tmName: [
          {
            required: true,
            message: "请输入品牌名称",
            trigger: "change"
          } /* validator校验器: 用于校验的回调函数 */ /* 值发生改变时触发 */ /* 失去焦点触发 */,
          /* { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' } */ {
            validator: this.validateTmName,
            trigger: "blur"
          }
        ],
        logoUrl: [{ required: true, message: "请指定LOGO图片" }]
      }
    };
  },
  mounted() {
    this.getTrademarks();
  },
  methods: {
    //校验品牌名称自定义校验函数
    validateTmName(rule, value, callback) {
      console.log("validateTmName()", value);
      if (value.length < 2 || value.length > 10) {
        callback(new Error("长度在 2 到 10 个字符"));
      } else {
        callback();
      }
    },
    // 上传图片
    handleLogoSuccess(res, file) {
      const logoUrl = res.data;
      this.form.logoUrl = logoUrl;
    },
    //指定图片限制
    beforeLogoUpload(file) {
      const isJPGOrPNG = ["image/jpeg", "image/png"].indexOf(file.type) > 0;
      const isLt500KB = file.size / 1024 < 500;
      if (!isJPGOrPNG) {
        this.$message.error("上传头像图片只能是 JPG/PNG 格式!");
      }
      if (!isLt500KB) {
        this.$message.error("上传头像图片大小不能超过500kb");
      }
      return isJPGOrPNG && isLt500KB;
    },

    //异步获取指定页面显示
    async getTrademarks(page = 1) {
      this.page = page;
      this.loading = true;
      const result = await this.$API.trademark.getList(page, this.limit);
      this.loading = false;
      if (result.code === 200) {
        const { records, total } = result.data;
        this.trademarks = records;
        this.total = total;
      } else {
        this.$message.error("获取品牌列表失败");
      }
    },
    //每页数量改变的监听回调
    handleSizeChange(pageSize) {
      this.limit = pageSize;
      this.getTrademarks();
    },
    //显示添加界面
    showAdd() {
      this.form = {
        tmName: "",
        logoUrl: ""
      };
      this.isShowDialog = true;
    },
    //显示修改界面
    showUpdate(trademark) {
      // this.form = trademark;
      this.form = { ...trademark };
      this.isShowDialog = true;
    },
    //添加或更新品牌
    async addOrUpdateTrademark() {
      const trademark = this.form;
      let result;
      if (trademark.id) {
        result = await this.$API.trademark.update(trademark);
      } else {
        result = await this.$API.trademark.add(trademark);
      }

      if (result.code === 200) {
        this.$message.success(`${trademark.id ? "更新" : "添加"}品牌成功`);
        this.isShowDialog = false;
        this.getTrademarks(trademark.id ? this.page : 1);
        this.form = {
          tmName: "",
          logoUrl: ""
        };
      } else {
        this.form = {
          tmName: "",
          logoUrl: ""
        };
        this.$message.error(`${trademark.id ? "更新" : "添加"}品牌失败`);
      }
    },
    //删除指定的品牌
    deleteTrademark(trademark) {
      this.$confirm(`确定删除 ${trademark.tmName} 吗?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          const result = await this.$API.trademark.remove(trademark.id);
          if (result.code === 200) {
            this.$message.success("删除品牌成功!");
            const page =
              this.trademarks.length === 1 ? this.page - 1 : this.page;
            this.getTrademarks(page);
          } else {
            this.$message.error("删除品牌失败!");
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
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
