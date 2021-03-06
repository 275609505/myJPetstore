<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span="6">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表 -->
      <el-table :data="orderList" stripe style="width: 75%">
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-form label-position="left" inline class="demo-table-expand">
              <el-form-item label="买家昵称">
                <span>{{ scope.row.username }}</span>
              </el-form-item>
              <el-form-item label="下单时间">
                <span>{{ scope.row.orderDate }}</span>
              </el-form-item>
              <el-form-item label="配送地址">
                <span>{{ scope.row.shipAddress1 }}{{ scope.row.shipAddress2 }}</span>
              </el-form-item>
            </el-form>

          </template>
        </el-table-column>
        <el-table-column label="订单编号" prop="orderId">
          <template slot-scope="scope">
            <el-link style="color: #0073ff" @click="showDetailDialog(scope.row.orderId)">{{scope.row.orderId}}</el-link>
          </template>
        </el-table-column>
        <el-table-column label="订单价格" prop="totalPrice"></el-table-column>
        <el-table-column label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" size="mini" v-if="scope.row.status === 'P'">已付款</el-tag>
            <el-tag type="danger" size="mini" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="sendstatus">
          <template slot-scope="scope">
            <el-tag type="success" size="mini" v-if="scope.row.sendstatus === 1">已发货</el-tag>
            <el-tag type="danger" size="mini" v-else>未发货</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.orderId)">修改地址</el-button>
            <el-button type="primary" size="mini" @click="showConfirmSendDialog(scope.row.orderId)" v-if="scope.row.sendstatus === 0" >点我发货</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 订单详细信息对话框 -->
    <el-dialog
      title="订单详细信息"
      :visible.sync="detailDialogVisible"
      width="50%"
      @close="detailDialogClosed"
    >
      <el-table :data="lineItemList" stripe style="width: 100%">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="itemId" prop="itemId"></el-table-column>
        <el-table-column label="单价" prop="unitPrice"></el-table-column>
        <el-table-column label="数量" prop="quantity"></el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="detailDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="detailDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 确认发货对话框 -->
    <el-dialog title="确认发货" :visible.sync="ConfirmSendDialogVisible" @close="confirmSendDialogDialogClosed" width="25%">
      <div style="text-align: center">
        <el-button @click="ConfirmSendDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="send">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改收货地址对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressDialogVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item label="address1" prop="address1">
          <el-input v-model="addressForm.address1"></el-input>
        </el-form-item>
        <el-form-item label="address2" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="changeAddr">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data() {
    return {
      // 订单列表查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      // 订单列表
      orderList: [],
      lineItemList: [],
      // 确认发货对话框
      ConfirmSendDialogVisible: false,
      // 修改地址对话框
      addressDialogVisible: false,
      // 查看订单详细信息对话框
      detailDialogVisible: false,
      addressForm: {
        id: '',
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      sendStatusForm: { // TODO
        orderId: 0,
        sendstatus: 0
      },
      cityData,
      // 物流进度对话框
      progressDialogVisible: false,
      // 物流进度
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('order/orders', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.status !== 200) {
        return this.$message.error('获取订单列表失败！')
      }
      // total表示这页显示多少订单
      this.total = res.data.total
      this.orderList = res.data.list
    },
    async getOrderItem(orderId) {
      const { data: res } = await this.$http.get('order/orderInfo', {
        params: { orderId: orderId }
      })
      console.log(res)
      this.lineItemList = res.data
    },
    // 分页
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newSize) {
      this.queryInfo.pagenum = newSize
      this.getOrderList()
    },
    showDetailDialog(orderId) {
      this.getOrderItem(orderId)
      this.detailDialogVisible = true
    },
    showEditDialog(id) {
      this.addressDialogVisible = true
      this.addressForm.id = id
    },
    showConfirmSendDialog(orderId) {
      this.sendStatusForm.orderId = orderId
      this.sendStatusForm.sendstatus = 1
      console.log(this.sendStatusForm)
      this.ConfirmSendDialogVisible = true
    },
    async send() {
      const qs = require('qs')
      const { data: res } = await this.$http.put('/order/sendstatus', qs.stringify(this.sendStatusForm))
      console.log(res)
      if (res.status !== 200) {
        return this.$message.error('发货失败!')
      }
      this.$message.success('发货成功！')
      this.ConfirmSendDialogVisible = false
      this.getOrderList()
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    detailDialogClosed() {
      this.$refs.detailRef.resetFields()
    },
    confirmSendDialogDialogClosed() {
      this.sendStatusForm = {}
    },
    async changeAddr() {
      console.log(this.addressForm)
      const qs = require('qs')
      const { data: res } = await this.$http.post('/order/changeAddr', qs.stringify(this.addressForm))
      console.log(this.addressForm)
      console.log(res)
      if (res.status !== 200) {
        return this.$message.error('修改发货地址失败')
      }
      this.addressDialogVisible = false
      this.$message.success('修改发货地址成功')
      this.getOrderList()
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 50%;
}
.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}
</style>
