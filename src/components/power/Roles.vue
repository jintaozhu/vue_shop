<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand" width="50">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom',index1===0?'bdtop':'','vcenter']"
              v-for="(item1,index1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag closable @close="removeRight(scope.row,item1.id)">{{item1.authName}}</el-tag>
              </el-col>
              <el-col :span="19">
                <el-row
                  :class="[index2===0?'':'bdtop','vcenter']"
                  v-for="(item2,index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      closable
                      type="success"
                      @close="removeRight(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      type="warning"
                      @close="removeRight(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#" width="50"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog
      title="分配权限"
      :visible.sync="showSetDialogVisible"
      width="50%"
      @close="monitorRightKeyOfDialogClosed"
    >
      <el-tree
        :data="rightList"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="leafRightKey"
        :props="treeRightProps"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showSetDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rolesList: [],
      showSetDialogVisible: false,
      rightList: [],
      treeRightProps: {
        children: "children",
        label: "authName"
      },
      leafRightKey: [],
      roleId: ""
    };
  },
  mounted() {
    this.getRolesList();
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      // console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error("请求角色列表失败");
      }
      this.rolesList = res.data;
    },
    async removeRight(role, rightId) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(error => error);
      if (confirmResult !== "confirm") {
        return this.$message.info("取消了删除");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除权限失败");
      }
      role.children = res.data;
    },
    async addRole() {
      const { data: res } = await this.$http.post("roles", {});
    },
    async showSetRightDialog(role) {
      this.roleId = role.id;
      this.leafRightKey = [];
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status !== 200) {
        return this.$message.error("请求权限列表失败");
      }
      this.rightList = res.data;
      this.getRightKey(role, this.leafRightKey);
      this.showSetDialogVisible = true;
    },
    getRightKey(role, leafRightKey) {
      if (!role.children) {
        return leafRightKey.push(role.id);
      }
      role.children.forEach(item => {
        // leafRightKey.push(item.id)
        this.getRightKey(item, leafRightKey);
      });
    },
    monitorRightKeyOfDialogClosed() {
      this.leafRightKey = [];
    },
    async allotRight() {
      const rightKeys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ];
      const ridStr = rightKeys.join(",");
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        {
          rids: ridStr
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("分配权限失败");
      }
      this.getRolesList();
      this.showSetDialogVisible = false;
    }
  }
};
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>