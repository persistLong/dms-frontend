<template>
  <a-card :bordered="false" class="card-area">
    <div :class="advanced ? 'search' : null">
      <!-- 搜索区域 -->
      <a-form layout="horizontal">
        <div :class="advanced ? null: 'fold'">
          <a-row >
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="设备编号"
                :labelCol="{span: 5}"
                :wrapperCol="{span: 18, offset: 1}">
                <a-input v-model="queryParams.code"/>
              </a-form-item>
            </a-col>
            <span style="float: right; margin-top: 3px;">
              <a-button type="primary" @click="search">查询</a-button>
              <a-button style="margin-left: 8px" @click="reset">重置</a-button>
            </span>
            <!--            <a-col :md="12" :sm="24" >-->
            <!--              <a-form-item-->
            <!--                label="创建时间"-->
            <!--                :labelCol="{span: 5}"-->
            <!--                :wrapperCol="{span: 18, offset: 1}">-->
            <!--                <range-date @change="handleDateChange" ref="createTime"></range-date>-->
            <!--              </a-form-item>-->
            <!--            </a-col>-->
          </a-row>
        </div>
      </a-form>
    </div>
    <div>
      <div class="operator" >
        <a-button v-hasPermission="'number:add'" type="primary" ghost @click="add">新增</a-button>
        <a-button v-hasPermission="'number:delete'" @click="batchDelete">删除</a-button>
        <!--        <a-dropdown v-hasPermission="'number:export'">-->
        <!--          <a-menu slot="overlay">-->
        <!--            <a-menu-item key="export-data" @click="exportExcel">导出Excel</a-menu-item>-->
        <!--          </a-menu>-->
        <!--          <a-button>-->
        <!--            更多操作 <a-icon type="down" />-->
        <!--          </a-button>-->
        <!--        </a-dropdown>-->
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
        <template slot="operation" slot-scope="text, record">
          <a-icon v-hasPermission="'number:update'" type="setting" theme="twoTone" twoToneColor="#4a9ff5" @click="edit(record)" title="修改"></a-icon>
          <a-badge v-hasNoPermission="'number:update'" status="warning" text="无权限"></a-badge>
        </template>
      </a-table>
    </div>
    <!-- 新增部门 -->
    <number-add
      @success="handleNumberAddSuccess"
      @close="handleNumberAddClose"
      :numberAddVisiable="numberAddVisiable">
    </number-add>
    <!-- 修改部门 -->
    <number-edit
      ref="numberEdit"
      @success="handleNumberEditSuccess"
      @close="handleNumberEditClose"
      :numberEditVisiable="numberEditVisiable">
    </number-edit>
  </a-card>
</template>

<script>
import RangeDate from '@/components/datetime/RangeDate'
import NumberAdd from './NumberAdd'
import NumberEdit from './NumberEdit'
export default {
  name: 'Number',
  components: {NumberAdd, NumberEdit, RangeDate},
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
      numberAddVisiable: false,
      numberEditVisiable: false
    }
  },
  computed: {
    columns () {
      return [{
        title: '设备编号',
        dataIndex: 'code'
      }, {
        title: '分类',
        dataIndex: 'categoryName'
      }, {
        title: '区域',
        dataIndex: 'companyBusinessName'
      }, {
        title: '位置',
        dataIndex: 'location'
      }, {
        title: '运行状态',
        dataIndex: 'status',
        customRender: (text, row, index) => {
          switch (text) {
            case '0':
              return <a-tag color="orange">离线</a-tag>
            case '1':
              return <a-tag color="green">在线</a-tag>
            default:
              return text
          }
        }
      }, {
        title: '负责人姓名',
        dataIndex: 'leaderName'
      }, {
        title: '负责人手机',
        dataIndex: 'leaderPhone'
      }, {
        title: '图片',
        dataIndex: 'url',
        customRender: (text, row, index) => {
          if (!text) {
            return <a-tag color="red">暂无图片</a-tag>
          }
          // 设备图片url
          let imgUrl = 'http://106.13.22.160/group1/' + text
          return <img style="width:40px;heigth:40px" slot="pic" slot-scope="text" src={imgUrl} />
        }
      }, {
        title: '操作',
        dataIndex: 'operation',
        scopedSlots: {customRender: 'operation'},
        fixed: 'right',
        width: 140
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
    handleNumberAddClose () {
      this.numberAddVisiable = false
    },
    handleNumberAddSuccess () {
      this.numberAddVisiable = false
      this.$message.success('新增设备成功')
      this.fetch()
    },
    add () {
      this.numberAddVisiable = true
    },
    handleNumberEditClose () {
      this.numberEditVisiable = false
    },
    handleNumberEditSuccess () {
      this.numberEditVisiable = false
      this.$message.success('修改设备成功')
      this.fetch()
    },
    edit (record) {
      this.numberEditVisiable = true
      this.$refs.numberEdit.setFormValues(record)
    },
    batchDelete () {
      if (!this.selectedRowKeys.length) {
        this.$message.warning('请选择需要删除的记录')
        return
      }
      let that = this
      let numberIds = []
      for (let key of that.selectedRowKeys) {
        numberIds.push(that.dataSource[key].numberId)
      }
      this.$confirm({
        title: '确定删除所选中的记录?',
        content: '当您点击确定按钮后，这些记录将会被彻底删除，如果其包含子记录，也将一并删除！',
        centered: true,
        onOk () {
          that.$delete('number/' + numberIds.join(',')).then(() => {
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
      // 重置分页
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
      // 将这三个参数赋值给Vue data，用于后续使用
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
      this.$get('number', {
        ...params
      }).then((r) => {
        let data = r.data
        this.loading = false
        this.dataSource = data.rows
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
