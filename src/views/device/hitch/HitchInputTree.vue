<template>
  <a-tree-select
    :allowClear="true"
    :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"
    :treeData="hitchTreeData"
    v-model="value">
  </a-tree-select>
</template>

<script>
export default {
  name: 'hitchInputTree',
  data () {
    return {
      hitchTreeData: [],
      value: undefined
    }
  },
  methods: {
    reset () {
      this.value = ''
    }
  },
  mounted () {
    this.$get('hitch').then((r) => {
      this.hitchTreeData = r.data.rows
    })
  },
  watch: {
    value (value) {
      this.$emit('change', value)
    }
  }
}
</script>

<style scoped>

</style>
