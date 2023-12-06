<template>
    <div v-loading="loading" class="ofa-container column">
      <div class="header">
        <div class="title-box">
          <span class="title ofa-text-title">消费日志<span class="sub-title" style="margin-left: 10px;">共有<label
                class="ofa-text-primary"> {{total}} </label>条数据</span></span>
        </div>
        <div class="search-box">
          <el-date-picker v-model="searchOption.startTime" type="datetime" placeholder="开始日期" format="yyyy-MM-dd"
            value-format="yyyy-MM-dd" size="small">
          </el-date-picker>
          <el-date-picker v-model="searchOption.endTime" type="datetime" placeholder="结束日期" format="yyyy-MM-dd"
            value-format="yyyy-MM-dd" size="small">
          </el-date-picker>
          <el-input v-model="searchOption.exChangeName" size="small" placeholder="交换机">
          </el-input>
          <el-input v-model="searchOption.routeKey" size="small" placeholder="路由名"></el-input>
          <el-input v-model="searchOption.queueName" size="small" placeholder="队列名"></el-input>
          <el-button type="primary" class="search-btn" size="small" @click="search">
            <font-awesome-icon fas icon="search"></font-awesome-icon>&nbsp;查询
          </el-button>
        </div>
      </div>
      <el-table :data="list" size="small" class="ofa-table">
        <el-table-column show-overflow-tooltip prop="Result" label="结果" width="100">
          <template slot-scope="scope">
            <label v-if="scope.row.Result === 'Success'" class="ofa-text-success" >{{ scope.row.Result }}</label>
            <label v-else-if="scope.row.Result.indexOf('Error') > -1 || scope.row.Result.indexOf('Fail') > -1" class="ofa-text-danger" >{{ scope.row.Result }}</label>
            <label v-else>{{ scope.row.Result }}</label>
          </template>
        </el-table-column>
        <el-table-column prop="ExChangeName" label="交换机" min-width="200"></el-table-column>
        <el-table-column prop="QueueName" label="队列名称" min-width="200"></el-table-column>
        <el-table-column prop="RouteKey" label="路由" min-width="200"></el-table-column>
        <el-table-column show-overflow-tooltip prop="OriginalMessage" label="原始消息" min-width="200"></el-table-column>
        <el-table-column prop="CreateTime" label="发送时间" width="160"></el-table-column>
        <el-table-column label="操作" width="120" align="center" header-align="center" fixed="right">
          <template slot-scope="scope">
            <el-button v-if="scope.row.Result.indexOf('Error') > -1 || scope.row.Result.indexOf('Fail') > -1" type="text" size="small" @click="retry(scope.row)">重新发送</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination background layout="total, sizes, prev, pager, next, jumper" :current-page="pageIndex"
        :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="total" @size-change="pageSizeChange"
        @current-change="pageIndexChange">
      </el-pagination>
    </div>
  </template>
  
  <script>
  import API from '../../../apis/ums-api'
  import { LOG } from '../../../router/ums-router'
  
  // 登录日志
  export default {
    name: LOG.name,
    data () {
      return {
        searchOption: {
          key: '', // 关键字
          startTime: '', // 开始日期
          endTime: '', // 结束日期
          exChangeName: '', // 交换机
          routeKey: '', // 路由
          queueName: '' // 队列名
        },
        loading: false, // 加载中
        list: [], // 需求列表
        total: 1, // 总数据量
        pageIndex: 1, // 页码
        pageSize: 10 // 页数
      }
    },
    beforeRouteEnter (to, from, next) {
      next(vm => vm.init())
    },
    methods: {
      init () {
        if (this.loading) return
        this.loading = true
        this.get()
      },
      get () {
        const url = this.$root.getApi(API.KEY, API.LOG.URL)
        this.axios.get(`${url}/${this.pageIndex}/${this.pageSize}`, {
            params: this.searchOption
          })
          .then(response => {
            this.total = response.Total
            this.list = response.Items
            this.loading = false
          })
      },
      pageSizeChange (value) {
        this.pageSize = value
        this.$nextTick(() => this.get())
      },
      pageIndexChange (value) {
        this.pageIndex = value
        this.$nextTick(() => this.get())
      },
      search () {
        this.pageIndex = 1
        this.$nextTick(() => this.get())
      },
      retry (row) {
        this.axios.post(row, row.OriginalMessage)
        .then(response => {
          if (response.Status) {
            this.get()
          }
        })
      }
    },
    created () {
      this.init()
    }
  }
  </script>
  
  <style lang="scss"scoped>
  .header {
    display: flex;
    flex-direction: column;
    height: auto !important;
    margin-bottom: 20px;
  
    .title-box {
      padding: 20px 0;
    }
  
    .title {
      font-size: 1.5rem;
      font-weight: 600;
    }
  
    .search-box {
  
      .el-input,
      .el-select,
      .el-date-editor {
        width: 200px;
        margin-right: 10px;
      }
    }
  
  }
  </style>
  