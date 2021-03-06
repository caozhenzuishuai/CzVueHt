<template>
  <div>
    <el-card>
      <CategorySelector ref="cs" @categoryChange="handleCategoryChange" />
    </el-card>

    <el-card>
      <div v-show="isShowList">
        <el-button
          type="primary"
          icon="el-icon-plus"
          style="margin-bottom: 20px"
          @click="showAdd"
          :disabled="!category3Id"
          >添加属性</el-button
        >

        <el-table border :data="attrs">
          <el-table-column
            label="序号"
            type="index"
            width="80"
            align="center"
          ></el-table-column>
          <el-table-column
            label="属性名称"
            width="150"
            prop="attrName"
          ></el-table-column>
          <el-table-column label="属性值列表">
            <template slot-scope="{ row, $index }">
              <el-tag
                style="margin: 2px"
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
                @click="showUpdate(row)"
              ></HintButton>
              <el-popconfirm
                :title="`确定删除 '${row.attrName}' 吗`"
                @onConfirm="deleteAttr(row.id)"
              >
                <hint-button
                  slot="reference"
                  title="删除"
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                />
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <div v-show="!isShowList">
        <el-form inline :model="attr">
          <el-form-item label="属性名">
            <el-input
              type="text"
              v-model="attr.attrName"
              placeholder="请输入属性名"
            ></el-input>
          </el-form-item>
        </el-form>

        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!attr.attrName"
          @click="addAttrValue"
          >添加属性值</el-button
        >
        <el-button @click="isShowList = true">取消</el-button>
        <el-table border style="margin: 20px 0" :data="attr.attrValueList">
          <el-table-column
            label="序号"
            type="index"
            width="80"
            align="center"
          ></el-table-column>
          <el-table-column label="属性值名称">
            <template slot-scope="{ row, $index }">
              <el-input
                :ref="$index"
                v-if="row.edit"
                v-model="row.valueName"
                size="mini"
                placeholder="请输入属性值名称"
                @blur="toShow(row)"
                @keyup.enter.native="toShow(row)"
              ></el-input>
              <span
                v-else
                @click="toEdit(row, $index)"
                style="display: inline-block; width: 100%"
              >
                {{ row.valueName }}
              </span>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除 '${row.valueName}' 吗`"
                @onConfirm="attr.attrValueList.splice($index, 1)"
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
        <el-button
          type="primary"
          @click="save"
          :disabled="!attr.attrName || attr.attrValueList.length === 0"
          >保存</el-button
        >
        <el-button @click="isShowList = true">取消</el-button>
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
      category1Id: "",
      category2Id: "",
      category3Id: "",
      attrs: [],

      isShowList: true,

      attr: {
        attrName: "",
        attrValueList: [],
        categoryId: "",
        categoryLevel: 3
      }
    };
  },

  mounted() {
    // this.category1Id = 2
    // this.category2Id = 13
    // this.category3Id = 61
    // this.getAttrs()
  },

  watch: {
    isShowList(value) {
      this.$refs.cs.disabled = !value;
    }
  },

  methods: {
    deleteAttr(id) {
      this.$API.attr
        .remove(id)
        .then(result => {
          this.getAttrs();
        })
        .catch(error => {
          this.$message.error("删除属性失败");
        });
    },
    async save() {
      const attr = this.attr;
      attr.attrValueList = attr.attrValueList.filter(value => {
        if (value.valueName !== "") {
          delete value.edit;
          return true;
        }
      });
      if (attr.attrValueList.length === 0) {
        this.$message.warning("至少指定一个属性值名称");
        return;
      }

      const result = await this.$API.attr.addOrUpdate(attr);
      if (result.code === 200) {
        this.$message.success("保存属性成功");
        this.isShowList = true;
        this.getAttrs();
      } else {
        this.$message.error("保存属性失败");
      }
    },
    toEdit(value, index) {
      if (value.hasOwnProperty("edit")) {
        value.edit = true;
      } else {
        this.$set(value, "edit", true);
      }
      this.$nextTick(() => {
        this.$refs[index].focus();
      });
    },
    toShow(value) {
      if (value.valueName) {
        const isRepeat = this.attr.attrValueList.some((item, index) => {
          if (item !== value) {
            return item.valueName === value.valueName;
          }
        });
        if (!isRepeat) {
          value.edit = false;
        } else {
          value.valueName = "";
          this.$message.warning("输入的名称已存在");
        }
      }
    },
    showAdd() {
      this.attr = {
        attrName: "",
        attrValueList: [],
        categoryId: this.category3Id,
        categoryLevel: 3
      };
      this.isShowList = false;
    },
    showUpdate(attr) {
      this.attr = cloneDeep(attr);
      this.isShowList = false;
    },
    addAttrValue() {
      this.attr.attrValueList.push({
        attrId: this.attr.id,
        valueName: "",
        edit: true
      });

      this.$nextTick(() => {
        this.$refs[this.attr.attrValueList.length - 1].focus();
      });
    },
    handleCategoryChange({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId;
        this.category2Id = "";
        this.category3Id = "";
        this.attrs = [];
      } else if (level === 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
        this.attrs = [];
      } else {
        this.category3Id = categoryId;
        this.getAttrs();
      }
    },
    async getAttrs() {
      const { category1Id, category2Id, category3Id } = this;
      const result = await this.$API.attr.getList(
        category1Id,
        category2Id,
        category3Id
      );
      this.attrs = result.data;
    }
  }
};
</script>

<style scoped>
.edit-input {
  padding-right: 60px;
}
.save-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
</style>
