<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getOrderList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrderList"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="tableData" border style="width: 100%">
        <el-table-column type="index" width="50"></el-table-column>
        <el-table-column prop="order_number" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column prop="order_pay" label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.order_pay === '1'"
              >已付款</el-tag
            >
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"></el-table-column>
        <el-table-column prop="create_time" label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column prop="address" label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showAddressDialog"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showLogicticsProgress"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改地址 -->

    <el-dialog
      title="修改地址"
      :visible.sync="addressVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        ref="addressFormRef"
        :model="addressForm"
        :rules="addressFormRules"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="provinces">
          <!-- <el-input v-model="addressForm.name"></el-input> -->
          <el-cascader
            v-model="addressForm.provinces"
            :options="dataCity"
            :props="{ expandTrigger: 'hover' }"
            @change="handleCascaderChange"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="detail">
          <el-input v-model="addressForm.detail"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <el-dialog title="物流进度" :visible.sync="logicticsVisible" width="50%">
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import dataCity from "./citydata";
export default {
  data() {
    return {
      tableData: [],
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      total: 0,
      addressVisible: false,
      addressForm: {
        provinces: [],
        detail: "",
      },
      addressFormRules: {
        provinces: [
          { required: true, message: "请选择省市区/县", trigger: "blur" },
        ],
        detail: [
          { required: true, message: "请填写详细地址", trigger: "blur" },
        ],
      },
      dataCity,
      logicticsVisible: false,
      progressInfo: [],
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      console.log(res, 111, this.queryInfo);
      this.tableData = res.data.goods;
      this.total = res.data.total;
    },
    handleSizeChange(val) {
      console.log(val, 222);
      this.queryInfo.pagesize = val;
      this.getOrderList();
    },
    handleCurrentChange(val) {
      console.log(val, 333);
      this.queryInfo.pagenum = val;
      this.getOrderList();
    },
    showAddressDialog() {
      this.addressVisible = true;
    },
    handleCascaderChange(val) {
      console.log(val);
      console.log(this.addressForm);
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields();
    },
    async showLogicticsProgress() {
      const { data: res } = await this.$http.get("/kuaidi/1106975712662");
      console.log(res);
      this.progressInfo = res.data;
      this.logicticsVisible = true;
    },
    // handleLogicticsClose() {
    //   this.progressInfo = [];
    //   console.log(this.progressInfo);
    // },
  },
};
</script>

<style>
.el-cascader {
  width: 100%;
}
</style>