<template>
  <a-card :bordered="false" class="card-area">
    <div>
      <div class="operator" >
        <a-button v-hasPermission="'notice:add'" type="primary" ghost @click="add">新增</a-button>
        <a-button v-hasPermission="'notice:delete'" @click="batchDelete">删除</a-button>
      </div>
      <!-- 表格区域 -->
      <a-table ref="TableInfo"
               :columns="columns"
               :dataSource="dataSource"
               :pagination="pagination"
               :loading="loading"
               :scroll="{ x: 900 }"
               :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
               @change="handleTableChange">
      </a-table>
    </div>
    <!-- 新增区域 -->
    <notice-add
      @success="handleNoticeAddSuccess"
      @close="handleNoticeAddClose"
      :noticeAddVisiable="noticeAddVisiable">
    </notice-add>
    <notice-edit
      ref="noticeEdit"
      @success="handleNoticeEditSuccess"
      @close="handleNoticeEditClose"
      :noticeEditVisiable="noticeEditVisiable">
    </notice-edit>
  </a-card>
</template>

<script>
import NoticeAdd from './NoticeAdd'
import NoticeEdit from './NoticeEdit'
import RangeDate from '@/components/datetime/RangeDate'
export default {
  name: 'Notice',
  components: {NoticeAdd, NoticeEdit, RangeDate},
  data () {
    return {
      advanced: false,
      dataSource: [],
      selectedRowKeys: [],
      queryParams: {},
      sortedInfo: null,
      paginationInfo: null,
      pagination: {
        pageSizeOptions: ['10', '20', '30', '40', '100'],
        defaultCurrent: 1,
        defaultPageSize: 10,
        showQuickJumper: true,
        showSizeChanger: true,
        showTotal: (total, range) => `显示 ${range[0]} ~ ${range[1]} 条记录，共 ${total} 条记录`
      },
      loading: false,
      noticeAddVisiable: false,
      noticeEditVisiable: false
    }
  },
  computed: {
    columns () {
      let {sortedInfo} = this
      sortedInfo = sortedInfo || {}
      return [{
        title: '序号',
        dataIndex: 'noticeId'
      }, {
        title: '公告描述',
        dataIndex: 'noticeDesc'
      }, {
        title: '创建时间',
        dataIndex: 'createTime',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'createTime' && sortedInfo.order
      }, {
        title: '操作',
        dataIndex: 'operation',
        scopedSlots: {customRender: 'operation'},
        fixed: 'right',
        width: 120
      }]
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    onSelectChange (selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    handleNoticeAddClose () {
      this.noticeAddVisiable = false
    },
    handleNoticeAddSuccess () {
      this.noticeAddVisiable = false
      this.$message.success('新增公告成功')
      this.fetch()
    },
    add () {
      this.noticeAddVisiable = true
    },
    handleNoticeEditClose () {
      this.noticeEditVisiable = false
    },
    handleNoticeEditSuccess () {
      this.noticeEditVisiable = false
      this.$message.success('修改终端成功')
      this.fetch()
    },
    edit (record) {
      this.noticeEditVisiable = true
      this.$refs.noticeEdit.setFormValues(record)
    },
    batchDelete () {
      if (!this.selectedRowKeys.length) {
        this.$message.warning('请选择需要删除的记录')
        return
      }
      let that = this
      this.$confirm({
        title: '确定删除所选中的记录?',
        content: '当您点击确定按钮后，这些记录将会被彻底删除，如果其包含子记录，也将一并删除！',
        centered: true,
        onOk () {
          that.$delete('notice/' + that.selectedRowKeys.join(',')).then(() => {
            that.$message.success('删除成功')
            that.selectedRowKeys = []
            that.fetch()
          })
        },
        onCancel () {
          that.selectedRowKeys = []
        }
      })
    },
    search () {
      let {sortedInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.fetch({
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams
      })
    },
    reset () {
      this.$refs.TableInfo.pagination.current = this.pagination.defaultCurrent
      if (this.paginationInfo) {
        this.paginationInfo.current = this.pagination.defaultCurrent
        this.paginationInfo.pageSize = this.pagination.defaultPageSize
      }
      // 取消选中
      this.selectedRowKeys = []
      // 重置列排序规则
      this.sortedInfo = null
      // 重置查询参数
      this.queryParams = {}
      // 清空时间选择
      this.$refs.createTime.reset()
      this.fetch()
    },
    handleTableChange (pagination, filters, sorter) {
      this.paginationInfo = pagination
      this.sortedInfo = sorter
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters
      })
    },
    fetch (params = {}) {
      this.loading = true
      if (this.paginationInfo) {
        // 如果分页信息不为空，则设置表格当前第几页，每页条数，并设置查询分页参数
        this.$refs.TableInfo.pagination.current = this.paginationInfo.current
        this.$refs.TableInfo.pagination.pageSize = this.paginationInfo.pageSize
        params.pageSize = this.paginationInfo.pageSize
        params.pageNum = this.paginationInfo.current
      } else {
        // 如果分页信息为空，则设置为默认值
        params.pageSize = this.pagination.defaultPageSize
        params.pageNum = this.pagination.defaultCurrent
      }
      this.$get('notice', {
        ...params
      }).then((r) => {
        let data = r.data
        this.loading = false
        this.dataSource = data.rows
        console.log(JSON.stringify(this.dataSource))
        const pagination = { ...this.pagination }
        pagination.total = data.total
        this.pagination = pagination
      })
    }
  }
}
</script>

<style scoped lang="less">
  @import "../../../../static/less/Common";
</style>
