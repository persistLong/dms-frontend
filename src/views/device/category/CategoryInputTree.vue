<template>
  <a-tree-select
    :allowClear="true"
    :dropdownStyle="{ maxHeight: '220px', overflow: 'auto' }"
    :treeData="categoryTreeData"
    v-model="value">
  </a-tree-select>
</template>

<script>
export default {
  name: 'CategoryInputTree',
  data () {
    return {
      categoryTreeData: [],
      value: undefined
    }
  },
  methods: {
    reset () {
      this.value = ''
    }
  },
  mounted () {
    this.$get('category').then((r) => {
      this.categoryTreeData = r.data.rows.children
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
