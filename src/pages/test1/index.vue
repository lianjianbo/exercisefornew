<template>
   <div>
       <div id="first">
        <el-radio v-model="radio" label="1">组播扫描</el-radio>
        <el-radio v-model="radio" label="2">指定扫描</el-radio>
        <el-button id="scan" v-on:click="myFunction" type="primary">{{mess}}</el-button>
        </div>
        <div>
           <el-button type="primary" plain>设备认证</el-button>
           <el-button type="danger">清空</el-button>
           <el-select class="search" v-model="value" multiple  filterable remote reserve-keyword
           placeholder="序列号/型号/IP" :remote-method="remoteMethod" :loading="loading">
           <el-option
           v-for="item in options"
           :key="item.value"
           :label="item.label"
           :value="item.value">
           </el-option>
           </el-select>
        </div>
  <el-table
       class="sn"
    ref="multipleTable"
    :data="tableData"
    tooltip-effect="dark"
    style="width: 5%"
    @selection-change="handleSelectionChange">
    <el-table-column
      class="select"
      type="selection"
      height="90"
      width="90">
    </el-table-column>
    </el-table>
      <el-table
      class="nav-get"
    :data="tableData"
    style="width: 70%"
    :default-sort = "{prop: 'date', order: 'descending'}">
    <el-table-column
      label="序列号"
      prop="SN"
      width="120">
    </el-table-column>
    <el-table-column
      prop="Type"
      label="类型"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="Version"
      label="固件版本"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="MAC"
      label="MAC地址"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="DHCP"
      label="DHCP"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="IP"
      label="IP地址"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="SubnetMask"
      label="子网掩码"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="Gateway"
      label="网关"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="DNS"
      label="DNS"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="Iden"
      label="认证"
      sortable
      width="120">
    </el-table-column>
    <el-table-column
      prop="State"
      label="状态"
      sortable
      width="120">
    </el-table-column>
    </el-table>
    <div class="basicset">
     <table ><i class="el-icon-d-arrow-right"></i>基本设置</table>
    </div>
    <div>
    <ul>
      <li class="list">序列号：</li>
      <li class="list">MAC地址：</li>
      <li class="list">固件版本：</li>
    </ul>
    </div>
<div>
  <el-switch class="swit"  v-model="value" inactive-text="DHCP"></el-switch>
  <form class="tab">
          <p>IP地址: <input type="text" name="fname" /></p>
          <p>子网掩码: <input type="text" name="lname" /></p>
          <p>网关: <input type="text" name="lname" /></p>
          <p>首选DNS: <input type="text" name="lname" /></p>
          <p>备选DNS: <input type="text" name="lname" /></p>
          <el-button type="primary" v-on:click="setting" class="set">设置</el-button>
        </form>
        </div>
        <div class="block">
    <span class="demonstration"></span>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage4"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="20"
      layout="total, sizes, prev, pager, next, jumper"
      :total="40">
    </el-pagination>
  </div>
   </div>
</template>
<script>
var mqtt = require('mqtt')
var client
const axios = require('axios')
export default {
  data () {
    return {
      mess: '开始扫描',
      client: null,
      radio: '1',
      value: false,
      multipleSelection: [],
      tableData: [{
        SN: '100000000001',
        Type: 'nohost',
        Version: 'v1.0.1',
        MAC: 'C1:8F:1A:69:23:5A',
        IP: '192.168.114.115',
        SubnetMask: '255.255.255.0',
        Gateway: '192.168.0.1',
        DNS: '8.8.8.8/114.114.114.114'},
      {
        SN: '100000000002',
        Type: 'nohost',
        Version: 'v1.0.1',
        MAC: 'C1:8F:1A:69:23:5A',
        IP: '192.168.114.115',
        SubnetMask: '255.255.255.0',
        Gateway: '192.168.0.1',
        DNS: '8.8.8.8/114.114.114.114'
      }],
      currentPage1: 5,
      currentPage2: 5,
      currentPage3: 5,
      currentPage4: 4
    }
  },
  methods: {
    toggleSelection (rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row)
        })
      } else {
        this.$refs.multipleTable.clearSelection()
      }
    },
    handleSelectionChange (val) {
      this.multipleSelection = val
    },
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
    },
    myFunction () {
      //  构造相关请求，发送给服务器，如果成功则按钮文字变为“停止扫描”
      //  接收设备发现通知，添加已发现的设备到页面列表
      var self = this
      axios.post('http://192.168.118.190:9001/api/devices/search/start')
        .then(function (response) {
          //  判断文字框里的文字和status的状态
          if (self.mess === '开始扫描' && response.data.status === 'success') {
            self.mess = '停止扫描'
          } else {
            axios.post('http://192.168.118.190:9001/api/devices/search/stop')
              .then(function (response) {
                if (self.mess === '停止扫描' && response.data.status === 'success') {
                  self.mess = '开始扫描'
                } else {
                  self.mess = '停止扫描'
                }
              })
          }
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    setting () {
      axios.patch('http://192.168.118.190:9001/apps/hispro/api/devices/:设备序列号SN/attributes')
        .then(function (response) {
          if (response.data.status === 'success') {
            alert('设置成功')
          } else {
            alert('设置失败')
          }
        })
    }
  },
  created () {
    // 编写 mqtt 连接逻辑
    client = mqtt.connect('ws://192.168.110.12:8083/mqtt')
    // 连接
    client.on('connect', function () {
      console.log('connected')
      // 订阅
      client.subscribe('/voerkadev/0/broadcast/discovering/device')
      // 发布
      client.publish('/voerkadev/0/broadcast/discovering/device', 'Hello mqtt')
      console.log('hello mqtt')
    })
  },
  beforeDestroy () {
    client.unsubscribe('/voerkadev/0/broadcast/discovering/device')
    // mqtt断开连接
    client.end()
  }
}
</script>
<style scoped>
#first{
  text-align: center;
  margin-left:-1000px;
  font-size: 15px;
}
.search{
text-align: center;
  margin-left: 820px;
}
.el-icon-d-caret{
  color: gray;
}
.basicset{
  margin-top: -200px;
  margin-left: 1390px;
}
.el-icon-d-arrow-right{
  color: skyblue;
}
.sn{
  margin-top: 55px;
}
.nav-get{
  margin-left: 60px;
  margin-top: -155px;
}
.set{
  margin-right: 50px;
}
.list{
  margin-left: 1700px;
  list-style:none;
}
.tab{
float: right;
margin-right:150px;
margin-top: 10%;
text-align:right;
position: relative;
}
.swit{
  margin-left: 1530px;
  margin-top:170px;
  position: absolute;
}
.block{
  margin-top: 500px;
}
</style>
