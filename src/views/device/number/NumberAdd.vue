<template>
  <a-drawer
    title="新增设备"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="numberAddVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='分类' v-bind="formItemLayout">
        <a-tree-select
          :allowClear="true"
          :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"
          :treeData="categoryTreeData"
          v-decorator="['categoryId',{rules: [{ required: true, message: '分类不能为空'}]}]"
          v-model="number.categoryId">
        </a-tree-select>
      </a-form-item>
      <a-form-item label='区域' v-bind="formItemLayout">
        <a-tree-select
          :allowClear="true"
          :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"
          :treeData="companyTreeData"
          v-decorator="['companyId',{rules: [{ required: true, message: '区域不能为空'}]}]"
          v-model="number.companyId">
        </a-tree-select>
      </a-form-item>
      <a-form-item label='设备编号'
                   v-bind="formItemLayout"
                   :validateStatus="validateStatus"
                   :help="help">
        <a-input v-model="number.code"
                 @blur="handleCodeBlur"
                 v-decorator="['code',{rules: [{ required: true, message: '设备编号不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='设备MAC地址'
                   v-bind="formItemLayout"
                   >
        <a-input v-model="number.mac"
                 v-decorator="['mac',{rules: [{ required: true, message: '设备MAC不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='位置' v-bind="formItemLayout">
        <a-input
          v-model="location"
          v-decorator="['location']" placeholder="请输入:省份＋城市＋区县＋城镇＋乡村＋街道＋门牌号码"/>
      </a-form-item>
    <!--  <a-form-item label='经度'
                   v-bind="formItemLayout">
        <a-input v-model="number.longitudes"
                 v-decorator="['longitudes', {rules: [{ message: '请输入正确的经度' }]}]"/>
      </a-form-item>
      <a-form-item label='纬度'
                   v-bind="formItemLayout"
                   >
        <a-input v-model="number.latitudes"
                 v-decorator="['latitudes']"/>
      </a-form-item>-->
<!--      <a-form-item label='密码' v-bind="formItemLayout">-->
<!--        <a-tooltip title='新用户默认密码为 1234qwer'>-->
<!--          <a-input type='password' readOnly :value="defaultPassword"/>-->
<!--        </a-tooltip>-->
<!--      </a-form-item>-->
    <!--&lt;!&ndash;  <a-form-item label='启用状态' v-bind="formItemLayout">-->
        <!--<a-radio-group-->
            <!--v-model="number.status"-->
            <!--v-decorator="['status', {rules: [{ required: true, message: '请选择启用状态' }]}]"-->
        <!--&gt;-->
          <!--<a-radio value="0">禁用</a-radio>-->
          <!--<a-radio value="1">启用</a-radio>-->
        <!--</a-radio-group>-->
        <!--</a-form-item>-->
      <!--<a-form-item label='运行状态' v-bind="formItemLayout">-->
        <!--<a-radio-group-->
          <!--v-model="number.pause"-->
          <!--v-decorator="['pause',{rules: [{ required: true, message: '请选择运行状态' }]}]"-->
        <!--&gt;-->
          <!--<a-radio value="0">在线</a-radio>-->
          <!--<a-radio value="1">离线</a-radio>-->
        <!--</a-radio-group>-->
      <!--</a-form-item>&ndash;&gt;-->
      <a-form-item label='负责人姓名' v-bind="formItemLayout">
        <a-input
          v-model="number.leaderName"
          v-decorator="['leaderName', { rules: [ {required: true, message: '负责人姓名不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='负责人电子邮件' v-bind="formItemLayout">
        <a-input
          v-model="number.leaderMail"
          v-decorator="['leaderMail',{rules: [
            { type: 'email', message: '请输入正确的邮箱' },
            { max: 50, message: '长度不能超过50个字符'},
            { required: true, message: '负责人电子邮件不能为空'}
          ]}]"/>
      </a-form-item>
      <a-form-item label="负责人手机" v-bind="formItemLayout">
        <a-input
          v-model="number.leaderPhone"
          v-decorator="['leaderPhone', {rules: [
            { pattern: '^0?(13[0-9]|15[012356789]|17[013678]|18[0-9]|14[57])[0-9]{8}$', message: '请输入正确的手机号'},
            { required: true, message: '负责人手机不能为空'}
          ]}]"/>
      </a-form-item>
      <a-form-item
        label="出售时间" v-bind="formItemLayout"
        >
<!--        :labelCol="{span: 5}"-->
<!--        :wrapperCol="{span: 18, offset: 1}"-->
        <a-date-picker @change="handleSalesDateChange" ref="salesTime"></a-date-picker>
      </a-form-item>
<!--      <a-form-item label='出售日期' v-bind="formItemLayout">-->
<!--        <a-input-->
<!--          v-model="number.salesTime"-->
<!--          v-decorator="['salesTime']"/>-->
<!--      </a-form-item>-->
      <a-form-item label='保修日期' v-bind="formItemLayout">
        <a-date-picker @change="handleGuaranteeDateChange" ref="guaranteeTime"></a-date-picker>
      </a-form-item>
      <a-form-item label="设备图片" v-bind="formItemLayout">
        <a-upload :multiple="false" :before-upload="beforeUpload" :fileList="fileList" :remove="handleRemove" :showUploadList="showUploadList" >
          <a-button>
            <a-icon type="upload"/>点击选择图片
          </a-button>
        </a-upload>
      </a-form-item>
<!--      <a-form-item label='角色' v-bind="formItemLayout">-->
<!--        <a-select-->
<!--          mode="multiple"-->
<!--          :allowClear="true"-->
<!--          v-model="user.roleId"-->
<!--          style="width: 100%"-->
<!--          v-decorator="['role',{rules: [{ required: true, message: '请选择角色' }]}]">-->
<!--          <a-select-option v-for="r in roleData" :key="r.roleId">{{r.roleName}}</a-select-option>-->
<!--        </a-select>-->
<!--      </a-form-item>-->
<!--      <a-form-item label='部门' v-bind="formItemLayout">-->
<!--        <a-tree-select-->
<!--          :allowClear="true"-->
<!--          :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"-->
<!--          :treeData="deptTreeData"-->
<!--          v-decorator="['deptId']"-->
<!--          v-model="user.deptId">-->
<!--        </a-tree-select>-->
<!--      </a-form-item>-->
<!--      <a-form-item label='状态' v-bind="formItemLayout">-->
<!--        <a-radio-group-->
<!--          v-model="user.status"-->
<!--          v-decorator="['status',{rules: [{ required: true, message: '请选择状态'}]}]">-->
<!--          <a-radio value="0">锁定</a-radio>-->
<!--          <a-radio value="1">有效</a-radio>-->
<!--        </a-radio-group>-->
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
// import RangeDate from '@/components/datetime/RangeDate'
import AFormItem from 'ant-design-vue/es/form/FormItem'
// import reqwest from 'reqwest'
// import store from '../../../store'
import axios from 'axios'
import ATextarea from 'ant-design-vue/es/input/TextArea'
const formItemLayout = {
  labelCol: { span: 5 },
  wrapperCol: { span: 15 }
}
export default {
  name: 'NumberAdd',
  components: {ATextarea, AFormItem},
  // components: { RangeDate },
  props: {
    numberAddVisiable: {
      default: false
    }
  },
  data () {
    return {
      number: {
        code: '',
        location: '',
        longitudes: '',
        latitudes: ''
      },
      loading: false,
      roleData: [],
      categoryTreeData: [],
      companyTreeData: [],
      location: '',
      timeout: null,
      formItemLayout,
      form: this.$form.createForm(this),
      validateStatus: '',
      help: '',
      fileList: [],
      uploading: false,
      showUploadList: {showPreviewIcon: true, showRemoveIcon: true}
    }
  },
  methods: {
    reset () {
      this.validateStatus = ''
      this.help = ''
      // this.user.username = ''
      this.number.code = ''
      this.loading = false
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleSubmit () {
      if (this.validateStatus !== 'success') {
        this.handleCodeBlur()
      }
      this.form.validateFields((err, values) => {
        if (!err && this.validateStatus === 'success') {
          this.loading = true
          let formData = new FormData()
          let number = this.number
          // create the formData from number
          for (let key in number) {
            formData.append(key, number[key])
          }
          this.$upload('number', formData).then((r) => {
            this.reset()
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    },
    handleCodeBlur () {
      let code = this.number.code.trim()
      let maxLength = 40
      let minLength = 1
      if (code.length) {
        if (code.length > maxLength) {
          this.validateStatus = 'error'
          this.help = `用户名不能超过${maxLength}个字符`
        } else if (code.length < minLength) {
          this.validateStatus = 'error'
          this.help = `用户名不能少于${minLength}个字符`
        } else {
          this.validateStatus = 'validating'
          this.$get(`number/check/${code}`).then((r) => {
            if (r.data) {
              this.validateStatus = 'success'
              this.help = ''
            } else {
              this.validateStatus = 'error'
              this.help = '抱歉，该设备编号已存在'
            }
          })
        }
      } else {
        this.validateStatus = 'error'
        this.help = '设备编号不能为空'
      }
    },
    handleGuaranteeDateChange (value, dateStr) {
      if (value) {
        var date = new Date(dateStr)
        this.number.guaranteeTime = date
      }
    },
    handleSalesDateChange (value, dateStr) {
      if (value) {
        var date = new Date(dateStr)
        this.number.salesTime = date
      }
    },
    beforeUpload (file) {
      let imgList = ['image/jpeg', 'image/png']
      if (imgList.indexOf(file.type) === -1) {
        this.$message.error('只支持jpg和png格式')
        return false
      }
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isLt2M) {
        this.$message.error('Image must smaller than 2MB!')
        return false
      }
      this.number.imgFile = file
      this.fileList = [file]
      return false
    },
    handleRemove (file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
    },
    getLocation (location) {
      axios.get('https://restapi.amap.com/v3/geocode/geo', {
        params: {
          address: location,
          key: '7d34f5c473caba282339693cab964295'
        }
      })
        .then((response) => {
          this.number.location = location
          this.number.longitudes = response.data.geocodes[0].location.split(',')[0]
          this.number.latitudes = response.data.geocodes[0].location.split(',')[1]
          if (this.number.longitudes !== null && this.number.latitudes !== null) {
            alert('获取经纬度成功！')
          }
        })
        .catch((error) => {
          console.log(error)
        })
    }
  },
  watch: {
    numberAddVisiable () {
      if (this.numberAddVisiable) {
        this.$get('company').then((r) => {
          this.companyTreeData = r.data.rows.children
        })
        this.$get('category').then((r) => {
          this.categoryTreeData = r.data.rows.children
        })
      }
    },
    location (curVal) {
      // 实现input连续输入，只发一次请求
      clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        this.getLocation(curVal)
      }, 400)
    }
  }

}
</script>

<style scoped>
#uploadContainer{
  display: flex;
  flex-direction: row;
}
</style>
