<template>
  <el-card>
    <el-table border style="width: 100%">
      <el-table-column label="序号" width="80"> </el-table-column>
      <el-table-column label="名称"> </el-table-column>
      <el-table-column label="描述"> </el-table-column>
      <el-table-column label="默认图片" width="150" align="center">
      </el-table-column>
      <el-table-column label="重量"> </el-table-column>
      <el-table-column label="价格"> </el-table-column>
      <el-table-column label="操作" width="250"> </el-table-column>
    </el-table>
  </el-card>
</template>

<script>
export default {
  name: "SkuList",
  data() {
    return {
      
    };
  },
  mounted() {
    const list = this.$API.sku.get(1);
    console.log("sku", list);
  }
};
</script>
