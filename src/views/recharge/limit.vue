<template>
  <div v-if="!editState" class="app-container calendar-list-container">

    <el-table :key='tableKey' :data="table" v-loading="listLoading" element-loading-text="给我一点时间" border fit highlight-current-row
              style="width: 100%">
      <el-table-column align="center" label="序号" width="50">
        <template slot-scope="scope">
          <span>{{scope.row.idnum}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" min-width="80" label="充值类型">
        <template slot-scope="scope">
          <span>{{scope.row.rechargetype}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" min-width="80" label="所在系统">
        <template slot-scope="scope">
          <span>会员系统</span>
        </template>
      </el-table-column>
      <el-table-column align="center" min-width="80" label="常用充值额度">
        <template slot-scope="scope">
          <span>{{scope.row.rechargeamount}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" min-width="80" label="付款金额">
        <template slot-scope="scope">
          <span>{{scope.row.payamount}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="操作" min-width="80" class-name="small-padding">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="memberlList_edit(scope.row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--<div v-show="!listLoading" class="pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.page" :page-size="listQuery.limit" layout="total, prev, pager, next" :total="total">
      </el-pagination>
    </div>-->

  </div>

  <div v-else-if="editState">
    <edit :data="editData" @edit="hideEditState"></edit>
  </div>
</template>

<script>
  import { fetchList, fetchPv } from '@/api/article'
  import waves from '@/directive/waves' // 水波纹指令
  import { parseTime } from '@/utils'
  import edit from './components/edit'
  const calendarTypeOptions = [
    { key: 'CN', display_name: '中国' },
    { key: 'US', display_name: '美国' },
    { key: 'JP', display_name: '日本' },
    { key: 'EU', display_name: '欧元区' }
  ]

  // arr to obj ,such as { CN : "中国", US : "美国" }
  const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
    acc[cur.key] = cur.display_name
    return acc
  }, {})

  export default {
    name: 'limit',
    components: { edit },
    directives: {
      waves
    },
    data() {
      return {
        table: [
          { idnum: 1, rechargetype: '储值', rechargeamount: '100', payamount: '100' },
          { idnum: 2, rechargetype: '储值', rechargeamount: '100', payamount: '100' },
          { idnum: 3, rechargetype: '储值', rechargeamount: '100', payamount: '100' },
          { idnum: 4, rechargetype: '储值', rechargeamount: '1000', payamount: '100' },
          { idnum: 5, rechargetype: '储值', rechargeamount: '1000', payamount: '100' },
          { idnum: 6, rechargetype: '储值', rechargeamount: '1000', payamount: '100' },
          { idnum: 7, rechargetype: '消费分', rechargeamount: '100', payamount: '100' },
          { idnum: 8, rechargetype: '消费分', rechargeamount: '100', payamount: '100' },
          { idnum: 9, rechargetype: '消费分', rechargeamount: '100', payamount: '100' },
          { idnum: 10, rechargetype: '消费分', rechargeamount: '100', payamount: '100' },
          { idnum: 11, rechargetype: '消费分', rechargeamount: '100', payamount: '100' },
          { idnum: 12, rechargetype: '消费分', rechargeamount: '100', payamount: '100' }
        ],
        tableKey: 0,
        list: null,
        total: null,
        listLoading: true,
        listQuery: {
          page: 1,
          limit: 10,
          importance: undefined,
          title: undefined,
          type: undefined,
          sort: '+id'
        },
        importanceOptions: [1, 2, 3],
        calendarTypeOptions,
        statusOptions: ['published', 'draft', 'deleted'],
        temp: {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          type: '',
          status: 'published'
        },
        dialogFormVisible: false,
        dialogStatus: '',
        textMap: {
          update: '编辑',
          create: '创建'
        },
        dialogPvVisible: false,
        pvData: [],
        rules: {
          type: [{ required: true, message: 'type is required', trigger: 'change' }],
          timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
          title: [{ required: true, message: 'title is required', trigger: 'blur' }]
        },
        editData: '',
        editState: false,
        adjustData: '',
        adjustState: false,
        detailData: '',
        detailState: false
      }
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          published: 'success',
          draft: 'info',
          deleted: 'danger'
        }
        return statusMap[status]
      },
      typeFilter(type) {
        return calendarTypeKeyValue[type]
      }
    },
    created() {
      this.getList()
    },
    methods: {
      memberlList_edit(val) {
        this.editData = val
        this.editState = true
      },
      hideEditState(data) {
        this.editState = false
      },
      memberlAccount_details(val) {
        this.detailData = val
        this.detailState = true
      },
      hideDetailState(val) {
        this.detailState = false
      },
      memberlAccount_adjust(val) {
        console.log(222)
        this.adjustData = val
        this.adjustState = true
      },
      hideAdjustState(val) {
        this.adjustState = false
      },
      getList() {
        this.listLoading = true
        fetchList(this.listQuery).then(response => {
          this.list = response.data.items
          this.total = response.data.total
          this.listLoading = false
        })
      },
      handleFilter() {
        this.listQuery.page = 1
        this.getList()
      },
      handleSizeChange(val) {
        this.listQuery.limit = val
        this.getList()
      },
      handleCurrentChange(val) {
        this.listQuery.page = val
        this.getList()
      },

      handleFetchPv(pv) {
        fetchPv(pv).then(response => {
          this.pvData = response.data.pvData
          this.dialogPvVisible = true
        })
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v => filterVal.map(j => {
          if (j === 'timestamp') {
            return parseTime(v[j])
          } else {
            return v[j]
          }
        }))
      }
    }
  }
</script>

