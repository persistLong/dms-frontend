<template>
  <a-drawer
    title="修改设备"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="numberEditVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='分类' v-bind="formItemLayout">
        <a-tree-select
          :allowClear="true"
          :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"
          :treeData="categoryTreeData"
          v-decorator="['categoryId',{rules: [{ required: true, message: '分类不能为空'}]}]"
        >
        </a-tree-select>
      </a-form-item>
      <a-form-item label='组织' v-bind="formItemLayout">
        <a-tree-select
          :allowClear="true"
          :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"
          :treeData="companyTreeData"
          v-decorator="['companyId',{rules: [{ required: true, message: '组织不能为空'}]}]"
        >
        </a-tree-select>
      </a-form-item>
      <a-form-item label='设备编号' v-bind="formItemLayout">
        <a-input v-decorator="['code',
                   {rules: [
                    { required: true, message: '设备编号不能为空'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='设备MAC地址' v-bind="formItemLayout">
        <a-input v-decorator="['mac',
                   {rules: [
                    { required: true, message: '设备MAC不能为空'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='经度' v-bind="formItemLayout">
        <a-input v-decorator="['longitudes']"/>
      </a-form-item>
      <a-form-item label='纬度' v-bind="formItemLayout">
        <a-input v-decorator="['latitudes']"/>
      </a-form-item>
      <a-form-item label='位置' v-bind="formItemLayout">
        <a-input v-decorator="['location']"/>
      </a-form-item>
      <a-form-item label='启用状态' v-bind="formItemLayout">
        <a-radio-group
          v-decorator="['status', {rules: [{ required: true, message: '请选择启用状态' }]}]"
        >
          <a-radio value="0">禁用</a-radio>
          <a-radio value="1">启用</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item label='运行状态' v-bind="formItemLayout">
        <a-radio-group
          v-decorator="['pause',{rules: [{ required: true, message: '请选择运行状态' }]}]"
        >
          <a-radio value="0">在线</a-radio>
          <a-radio value="1">离线</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item label='负责人姓名' v-bind="formItemLayout">
        <a-input
          v-decorator="['leaderName', { rules: [ {required: true, message: '负责人姓名不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label="负责人手机" v-bind="formItemLayout">
        <a-input
          v-decorator="['leaderPhone', {rules: [
            { pattern: '^0?(13[0-9]|15[012356789]|17[013678]|18[0-9]|14[57])[0-9]{8}$', message: '请输入正确的手机号'},
            { required: true, message: '负责人手机不能为空'}
          ]}]"/>
      </a-form-item>
      <a-form-item label='负责人电子邮件' v-bind="formItemLayout">
        <a-input
          v-decorator="['leaderMail',{rules: [
            { type: 'email', message: '请输入正确的邮箱' },
            { max: 50, message: '长度不能超过50个字符'},
            { required: true, message: '负责人电子邮件不能为空'}
          ]}]"/>
      </a-form-item>
      <a-form-item
        label="出售时间" v-bind="formItemLayout"
      >
        <!--        :labelCol="{span: 5}"-->
        <!--        :wrapperCol="{span: 18, offset: 1}"-->
        <a-date-picker :value="salesTime" ref="salesTime" :defaultValue="number.salesTime"></a-date-picker>
      </a-form-item>
      <a-form-item
        label="保修时间" v-bind="formItemLayout"
      >
        <!--        :labelCol="{span: 5}"-->
        <!--        :wrapperCol="{span: 18, offset: 1}"-->
        <a-date-picker :value="guaranteeTime" ref="salesTime"></a-date-picker>
      </a-form-item>
    </a-form>
    <div class="drawer-bootom-button">
      <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="loading">提交</a-button>
    </div>
  </a-drawer>
</template>

<script>
const formItemLayout = {
  labelCol: { span: 5 },
  wrapperCol: { span: 15 }
}
export default {
  name: 'NumberEdit',
  props: {
    numberEditVisiable: {
      default: false
    }
  },
  data () {
    return {
      loading: false,
      formItemLayout,
      form: this.$form.createForm(this),
      // categoryTreeKey: +new Date(),
      // companyTreeKey: '',
      number: {},
      checkedKeys: [],
      expandedKeys: [],
      defaultCheckedCompanyKeys: [],
      defaultCheckedCategoryKeys: [],
      categoryTreeData: [],
      companyTreeData: [],
      salesTime: '',
      guaranteeTime: ''
    }
  },
  methods: {
    reset () {
      this.loading = false
      // this.companyTreeKey = +new Date()
      // this.categoryTreeKey = +new Date()
      this.expandedKeys = this.checkedKeys = this.defaultCheckedKeys = []
      this.button = {}
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleCheck (checkedKeys) {
      this.checkedKeys = checkedKeys
    },
    handleExpand (expandedKeys) {
      this.expandedKeys = expandedKeys
    },
    setFormValues ({...number}) {
      this.form.getFieldDecorator('code')
      this.form.setFieldsValue({'code': number.code})
      this.form.getFieldDecorator('mac')
      this.form.setFieldsValue({'mac': number.mac})
      this.form.getFieldDecorator('longitudes')
      this.form.setFieldsValue({'longitudes': number.longitudes})
      this.form.getFieldDecorator('latitudes')
      this.form.setFieldsValue({'latitudes': number.latitudes})
      this.form.getFieldDecorator('leaderPhone')
      this.form.setFieldsValue({'leaderPhone': number.leaderPhone})
      this.form.getFieldDecorator('leaderName')
      this.form.setFieldsValue({'leaderName': number.leaderName})
      this.form.getFieldDecorator('leaderMail')
      this.form.setFieldsValue({'leaderMail': number.leaderMail})
      this.form.getFieldDecorator('status')
      this.form.setFieldsValue({'status': number.status})
      this.form.getFieldDecorator('pause')
      this.form.setFieldsValue({'pause': number.pause})
      this.form.getFieldDecorator('categoryId')
      this.form.setFieldsValue({'categoryId': number.categoryId})
      this.form.getFieldDecorator('companyId')
      this.form.setFieldsValue({'companyId': number.companyId})
      this.form.getFieldDecorator('numberId')
      this.form.getFieldDecorator('salesTime')
      this.form.setFieldsValue({'salesTime': number.salesTime})
      this.form.getFieldDecorator('guaranteeTime')
      this.form.setFieldsValue({'guaranteeTime': number.guaranteeTime})
      // console.log('----', number.guaranteeTime)
      this.number.numberId = number.numberId
    },
    handleSubmit () {
      // let checkedArr = Object.is(this.checkedKeys.checked, undefined) ? this.checkedKeys : this.checkedKeys.checked
      // if (checkedArr.length > 1) {
      //   this.$message.error('最多只能选择一个上级部门，请修改')
      //   return
      // }
      // if (checkedArr[0] === this.number.numberId) {
      //   this.$message.error('不能选择自己作为上级部门，请修改')
      //   return
      // }
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          let number = this.form.getFieldsValue()
          // number.parentId = checkedArr[0]
          // if (Object.is(number.parentId, undefined)) {
          //   number.parentId = 0
          // }
          number.numberId = this.number.numberId
          this.$put('number', {
            ...number
          }).then(() => {
            this.reset()
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    }
  },
  watch: {
    numberEditVisiable () {
      if (this.numberEditVisiable) {
        this.$get('company').then((r) => {
          this.companyTreeData = r.data.rows.children
          // this.numberTreeKey = +new Date()
        })
        this.$get('category').then((r) => {
          this.categoryTreeData = r.data.rows.children
          // this.numberTreeKey = +new Date()
        })
      }
    }
  }
}
</script>

<style scoped>

</style>
