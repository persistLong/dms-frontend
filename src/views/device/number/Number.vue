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
                <a-input v-model="queryParams.numberName"/>
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
      <a-table :columns="columns"
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
      pagination: {
        defaultPageSize: 10000000,
        hideOnSinglePage: true,
        indentSize: 100
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
        title: '组织',
        dataIndex: 'companyBusinessName'
      }, {
        title: '位置',
        dataIndex: 'location'
      }, {
        title: '启用状态',
        dataIndex: 'status'
      }, {
        title: '运行状态',
        dataIndex: 'pause'
      }, {
        title: '负责人姓名',
        dataIndex: 'leaderName'
      }, {
        title: '负责人手机',
        dataIndex: 'leaderPhone'
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
      this.$get('number', {
        ...params
      }).then((r) => {
        let data = r.data
        this.loading = false
        this.dataSource = data.rows
      })
    }
  }
}
</script>

<style scoped lang="less">
  @import "../../../../static/less/Common";
</style>
