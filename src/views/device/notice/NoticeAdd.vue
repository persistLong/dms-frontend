<template>
  <a-drawer
    title="新增公告"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="noticeAddVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='公告内容' v-bind="formItemLayout">
        <a-input
                 v-decorator="['noticeDesc',
                   {rules: [
                    { required: true, message: '公告内容不能为空'},
                    { max: 40, message: '公告内容不能超过40个字符'}
                  ]}]"/>
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
  name: 'NoticeAdd',
  props: {
    noticeAddVisiable: {
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
      this.notice = {}
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleSubmit () {
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          this.$post('notice', {
            ...this.notice
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
    noticeAddVisiable () {
      if (this.noticeAddVisiable) {
        this.$get('notice').then((r) => {
          this.notice = r.data.rows.children
        })
      }
    }
  }
}
</script>

<style scoped>

</style>
