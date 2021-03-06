<template>
  <d2-container>
    <!-- header 查询条件 -->
    <template slot="header">
      <el-form
        :inline="true"
        :model="listQuery"
        size="mini"
        style="margin-bottom: -18px;">
          <el-form-item label="日志类型" prop="type">
            <el-select v-model="listQuery.type" filterable placeholder="请选择" clearable>
              <el-option v-for="item in dicts" :key="item.value" :label="item.label" :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="default" icon="el-icon-search" @click="handleFilter">搜 索</el-button>
          </el-form-item>
      </el-form>
    </template>
    <el-table
            :key='tableKey'
            :data="list"
            v-loading="listLoading"
            element-loading-text="拼命加载中..."
            highlight-current-row
            stripe
            style="width: 100%">
      <el-table-column align="center" label="序号" width="60">
        <template slot-scope="scope">
          <span>{{ getSerialNumber(scope.$index) }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="类型">
        <template slot-scope="scope">
          <span>
            <el-button type="success" v-if="scope.row.type == 0">{{ scope.row.type | typeFilter }}</el-button>
            <el-button type="danger" v-if="scope.row.type ==9">{{ scope.row.type | typeFilter }}</el-button>
          </span>
        </template>
      </el-table-column>

      <el-table-column label="请求接口" show-overflow-tooltip>
        <template slot-scope="scope">
          <span>{{ scope.row.requestUri }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="IP地址">
        <template slot-scope="scope">
          <span>{{ scope.row.remoteAddr }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="请求方式">
        <template slot-scope="scope">
          <span>{{ scope.row.method }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="传入参数" show-overflow-tooltip>
        <template slot-scope="scope">
          <span>{{ scope.row.params }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="请求时间">
        <template slot-scope="scope">
          <span>{{ scope.row.time}}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="创建时间">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime | moment('YYYY-MM-DD HH:mm') }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="danger" v-if="sys_log_del" @click="handleDelete(scope.row)" icon="el-icon-delete"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- footer 分页条 -->
    <template slot="footer">
        <el-pagination
          background
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="listQuery.page"
          :page-sizes="[10,20,30,50]"
          :page-size="listQuery.limit"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          style="margin: -10px;">
        </el-pagination>
    </template>
  </d2-container>
</template>

<script>
import { delObj, fetchList } from '@/api/log'
import { remote } from '@/api/dict'
import { mapGetters } from 'vuex'

export default {
  name: 'table_log',
  data () {
    return {
      list: null,
      total: null,
      sys_dict_add: false,
      listLoading: true,
      dicts: [],
      listQuery: {
        page: 1,
        limit: 10,
        type: undefined
      },
      tableKey: 0
    }
  },
  computed: {
    ...mapGetters(['permissions'])
  },
  filters: {
    typeFilter (type) {
      const typeMap = {
        0: '正常',
        9: '异常'
      }
      return typeMap[type]
    }
  },
  created () {
    this.getList()
    this.sys_log_del = this.permissions['sys_log_del']
    remote('log_type').then(response => {
      this.dicts = response.data
    })
  },
  methods: {
    getSerialNumber (index) {
      return index + 1 + (this.listQuery.page - 1) * this.listQuery.limit
    },
    getList () {
      this.listLoading = true
      this.listQuery.orderByField = 'create_time'
      this.listQuery.isAsc = false
      fetchList(this.listQuery).then(response => {
        this.list = response.data.records
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleSizeChange (val) {
      this.listQuery.limit = val
      this.getList()
    },
    handleCurrentChange (val) {
      this.listQuery.page = val
      this.getList()
    },
    handleDelete (row) {
      delObj(row.id).then(response => {
        this.dialogFormVisible = false
        this.getList()
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    handleFilter () {
      this.listQuery.page = 1
      this.getList()
    }
  }
}
</script>
<style lang="scss" scoped>

</style>
