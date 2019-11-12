<template>
  <a-drawer
    title="修改公告"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="noticeEditVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='公告内容' v-bind="formItemLayout">
        <a-input v-decorator="['noticeDesc',
                 {rules: [
                 { required: true, message: '公告内容不能为空'},
        { max: 40, message: '公告内容不能超过40个字符'}
        ]}]"/>
      </a-form-item>
<!--      <a-form-item label='分类排序' v-bind="formItemLayout">-->
<!--        <a-input-number v-decorator="['orderNum']" style="width: 100%"/>-->
<!--      </a-form-item>-->
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
  name: 'NoticeEdit',
  props: {
    noticeEditVisiable: {
      default: false
    }
  },
  data () {
    return {
      loading: false,
      formItemLayout,
      form: this.$form.createForm(this),
      notice: {}
    }
  },
  methods: {
    reset () {
      this.loading = false
      this.button = {}
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    setFormValues ({...notice}) {
      this.form.getFieldDecorator('noticeDesc')
      this.form.setFieldsValue({'noticeDesc': notice.text})
      this.notice.noticeId = notice.id
    },
    handleSubmit () {
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          let notice = this.form.getFieldsValue()
          notice.noticeId = this.notice.noticeId
          this.$put('notice', {
            ...notice
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
    noticeEditVisiable () {
      if (this.noticeEditVisiable) {
        this.$get('notice').then((r) => {
        })
      }
    }
  }
}
</script>

<style scoped>

</style>
