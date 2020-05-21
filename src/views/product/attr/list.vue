<template>
  <div>
    <el-card>
      <CategorySelector @categoryChange="handleCategoryChange" ref="cs" />
    </el-card>

    <el-card style="margin-top: 20px">
      <div v-show="isShowList">
        <el-button
          type="primary"
          icon="el-icon-plus"
          style="margin-bottom: 20px"
          @click="toAddAttr"
          :disabled="category3Id === null"
          >添加属性</el-button
        >

        <el-table border :data="attrList">
          <el-table-column
            label="序号"
            type="index"
            width="100"
            align="center"
          ></el-table-column>

          <el-table-column
            label="名称"
            width="200"
            prop="attrName"
          ></el-table-column>

          <el-table-column label="属性值列表">
            <template slot-scope="{ row }">
              <el-tag
                type="info"
                v-for="value in row.attrValueList"
                :key="value.id"
                >{{ value.valueName }}</el-tag
              >
            </template>
          </el-table-column>

          <el-table-column label="操作" width="150">
            <template slot-scope="{ row, $index }">
              <HintButton
                title="修改"
                type="primary"
                icon="el-icon-edit"
                size="mini"
                @click="toUpdateAttr(row)"
              ></HintButton>
              <el-popconfirm
                :title="`确定删除属性 ${row.attrName} 吗?`"
                @onConfirm="deleteAttr(row.id)"
              >
                <HintButton
                  slot="reference"
                  title="删除"
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                ></HintButton>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div v-show="!isShowList">
        <el-form inline :model="attr">
          <el-form-item label="属性名称">
            <el-input type="text" v-model="attr.attrName" />
          </el-form-item>
        </el-form>

        <div style="margin-bottom: 20px">
          <el-button
            type="primary"
            icon="el-icon-plus"
            @click="addValue"
            :disabled="!attr.attrName.trim()"
            >添加属性值</el-button
          >
          <el-button @click="isShowList = true">取消</el-button>
        </div>

        <el-table border :data="attr.attrValueList">
          <el-table-column
            label="序号"
            type="index"
            width="100"
            align="center"
          ></el-table-column>

          <el-table-column label="属性值名称">
            <template slot-scope="{ row, $index }">
              <el-input
                type="text"
                size="mini"
                placeholder="输入属性值按enter确定"
                v-model="row.valueName"
                v-if="row.edit"
                @blur="toViewValue(row)"
                @keyup.enter.native="toViewValue(row)"
              ></el-input>
              <span
                v-else
                @click="toEditValue(row)"
                style="display: inline-block;width: 100%;"
                >{{ row.valueName }}</span
              >
            </template>
          </el-table-column>

          <el-table-column label="操作">
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除属性值 ${row.valueName} 吗?`"
                @onConfirm="deleteValue($index)"
              >
                <HintButton
                  slot="reference"
                  title="删除"
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                ></HintButton>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>

        <div style="margin: 20px 0">
          <el-button
            type="primary"
            @click="addUpdateAttr"
            :disabled="!isSaveValid"
            >保存</el-button
          >
          <el-button @click="isShowList = true">取消</el-button>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "AttrList",

  data() {
    return {
      attrList: [],
      category1Id: null,
      category2Id: null,
      category3Id: null,
      isShowList: true,

      attr: {
        attrName: "",
        attrValueList: []
      }
    };
  },

  computed: {
    isSaveValid() {
      return (
        this.attr.attrName.trim() &&
        this.attr.attrValueList.some(item => !!item.valueName.trim())
      );
    }
  },

  watch: {
    isShowList(value) {
      this.$refs.cs.isDisabled = !value;
    }
  },

  methods: {
    handleCategoryChange({ categoryId, level }) {
      console.log("handleCategoryChange", categoryId, level);
      if (level === 3) {
        this.category3Id = categoryId;
        this.getList();
      } else if (level === 1) {
        this.category1Id = categoryId;
        this.category2Id = null;
        this.category3Id = null;
        this.attrList = [];
      } else {
        this.category2Id = categoryId;
        this.category3Id = null;
        this.attrList = [];
      }
    },

    async getList() {
      const result = await this.$API.attr.getList(
        this.category1Id,
        this.category2Id,
        this.category3Id
      );
      this.attrList = result.data;
    },

    toUpdateAttr(attr) {
      this.attr = cloneDeep(attr);
      this.isShowList = false;
    },
    toAddAttr() {
      this.attr = {
        attrName: "",
        attrValueList: [],
        categoryId: this.category3Id,
        categoryLevel: 3
      };
      this.isShowList = false;
    },
    deleteValue(index) {
      this.attr.attrValueList.splice(index, 1);
    },

    addValue() {
      this.attr.attrValueList.push({
        valueName: "",
        attrId: this.attr.id,
        edit: true
      });
    },
    toViewValue(value) {
      if (value.valueName) {
        value.edit = false;
      }
    },
    toEditValue(value) {
      if (!value.hasOwnProperty("edit")) {
        this.$set(value, "edit", true);
      } else {
        value.edit = true;
      }
    },

    async addUpdateAttr() {
      this.attr.attrValueList = this.attr.attrValueList.filter(value => {
        delete value.edit;
        return !!value.valueName.trim();
      });
      const result = await this.$API.attr.addOrUpdate(this.attr);
      if (result.code === 200) {
        this.$message.success(`${this.attr.id ? "更新" : "添加"}属性成功`);
        this.isShowList = true;
        this.getList();
      } else {
        this.$message.error(`${this.attr.id ? "更新" : "添加"}属性失败`);
      }
    },
    async deleteAttr(id) {
      const result = await this.$API.attr.remove(id);
      if (result.code === 200) {
        this.$message.success(`删除属性成功`);
        this.getList();
      } else {
        this.$message.error(`删除属性失败`);
      }
    }
  }
};
</script>

<style scoped></style>
