<template>
  <div>
    <el-button @click="showAddCateDialog">hover 触发子菜单</el-button>
    <!-- <el-cascader
      v-model="value"
      :options="options"
      :props="{ expandTrigger: 'hover' }"
    ></el-cascader>-->
    <el-dialog title="提示" :visible.sync="addCateCenterDialogVisible" width="50%">
      <el-form  ref="ruleFormRef" label-width="100px">
        <el-form-item label="分类名称">
          <el-input></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            style="width:100%"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateCenterDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateCenterDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      addCateCenterDialogVisible: false,
      parentCateList: [],
      cascaderProps: {
        expandTrigger: "hover",
        value: "cat_id",
        label: "cat_name",
        children: "children"
      },

      selectedKeys: []
    };
  },
  methods: {
    showAddCateDialog() {
      this.getParentCateList();
      this.addCateCenterDialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 }
      });
      if (res.meta.status !== 200) {
        return this.$message.error("请求父级列表失败");
      }
      console.log(res)
      this.parentCateList = res.data;
      console.log(this.parentCateList);
    }
  }
};
</script>