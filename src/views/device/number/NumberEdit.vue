<template>
  <a-drawer
    title="修改按钮"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="numberEditVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='部门名称' v-bind="formItemLayout">
        <a-input v-decorator="['numberName',
                   {rules: [
                    { required: true, message: '部门名称不能为空'},
                    { max: 20, message: '长度不能超过20个字符'}
                  ]}]"/>
      </a-form-item>
<!--      <a-form-item label='部门排序' v-bind="formItemLayout">-->
<!--        <a-input-number v-decorator="['orderNum']" style="width: 100%"/>-->
<!--      </a-form-item>-->
      <a-form-item label='上级部门'
                   style="margin-bottom: 2rem"
                   v-bind="formItemLayout">
        <a-tree
          :key="numberTreeKey"
          :checkable="true"
          :checkStrictly="true"
          @check="handleCheck"
          @expand="handleExpand"
          :expandedKeys="expandedKeys"
          :defaultCheckedKeys="defaultCheckedKeys"
          :treeData="numberTreeData">
        </a-tree>
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
      numberTreeKey: +new Date(),
      number: {},
      checkedKeys: [],
      expandedKeys: [],
      defaultCheckedKeys: [],
      numberTreeData: []
    }
  },
  methods: {
    reset () {
      this.loading = false
      this.numberTreeKey = +new Date()
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
      this.form.getFieldDecorator('numberName')
      this.form.setFieldsValue({'numberName': number.text})
      this.form.getFieldDecorator('orderNum')
      this.form.setFieldsValue({'orderNum': number.order})
      if (number.parentId !== '0') {
        this.defaultCheckedKeys.push(number.parentId)
        this.checkedKeys = this.defaultCheckedKeys
        this.expandedKeys = this.checkedKeys
      }
      this.number.numberId = number.id
    },
    handleSubmit () {
      let checkedArr = Object.is(this.checkedKeys.checked, undefined) ? this.checkedKeys : this.checkedKeys.checked
      if (checkedArr.length > 1) {
        this.$message.error('最多只能选择一个上级部门，请修改')
        return
      }
      if (checkedArr[0] === this.number.numberId) {
        this.$message.error('不能选择自己作为上级部门，请修改')
        return
      }
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          let number = this.form.getFieldsValue()
          number.parentId = checkedArr[0]
          if (Object.is(number.parentId, undefined)) {
            number.parentId = 0
          }
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
        this.$get('number').then((r) => {
          this.numberTreeData = r.data.rows.children
          this.numberTreeKey = +new Date()
        })
      }
    }
  }
}
</script>

<style scoped>

</style>
