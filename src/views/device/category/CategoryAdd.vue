<template>
  <a-drawer
    title="新增终端"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="categoryAddVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='类别名称' v-bind="formItemLayout">
        <a-input v-model="category.categoryName"
                 v-decorator="['categoryName',
                   {rules: [
                    { required: true, message: '类别名称不能为空'},
                    { max: 20, message: '长度不能超过20个字符'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='类别排序' v-bind="formItemLayout">
        <a-input-number v-model="category.orderNum" style="width: 100%"/>
      </a-form-item>
      <a-form-item label='上级类别'
                   style="margin-bottom: 2rem"
                   v-bind="formItemLayout">
        <a-tree
          :key="categoryTreeKeys"
          :checkable="true"
          :checkStrictly="true"
          @check="handleCheck"
          @expand="handleExpand"
          :expandedKeys="expandedKeys"
          :treeData="categoryTreeData">
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
  labelCol: { span: 3 },
  wrapperCol: { span: 18 }
}
export default {
  name: 'CategoryAdd',
  props: {
    categoryAddVisiable: {
      default: false
    }
  },
  data () {
    return {
      loading: false,
      formItemLayout,
      form: this.$form.createForm(this),
      category: {},
      checkedKeys: [],
      expandedKeys: [],
      categoryTreeData: [],
      categoryTreeKeys: +new Date()
    }
  },
  methods: {
    reset () {
      this.loading = false
      this.categoryTreeKeys = +new Date()
      this.expandedKeys = this.checkedKeys = []
      this.category = {}
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
    handleSubmit () {
      let checkedArr = Object.is(this.checkedKeys.checked, undefined) ? this.checkedKeys : this.checkedKeys.checked
      if (checkedArr.length > 1) {
        this.$message.error('最多只能选择一个上级类别，请修改')
        return
      }
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          if (checkedArr.length) {
            this.category.parentId = checkedArr[0]
          } else {
            this.category.parentId = ''
          }
          this.$post('category', {
            ...this.category
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
    categoryAddVisiable () {
      if (this.categoryAddVisiable) {
        this.$get('category').then((r) => {
          this.categoryTreeData = r.data.rows.children
        })
      }
    }
  }
}
</script>

<style scoped>

</style>
