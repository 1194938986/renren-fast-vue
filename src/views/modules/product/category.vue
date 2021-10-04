<!--
  功能：功能描述
  作者：XueQi
  邮箱：1194938986@qq.com
  时间：2021年09月18日 20:31:13
  版本：v1.0
  修改记录：
  修改内容：
  修改人员：
  修改时间：
-->
<template>
  <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId">
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span> {{ node.label }}</span>
      <span>
        <el-button v-if="node.level" type="text" size="mini" @click="() => append(data)">
          Append
        </el-button>
        <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)">
          Delete
        </el-button>
      </span>
    </span>
  </el-tree>
</template>

<script>
export default {
  // 组件名称
  name: `demo`,
  // 组件参数 接收来自父组件的数据
  props: {},
  // 局部注册的组件
  components: {},

  data () {
    return {
      expandedKey: [2],
      menus: [],
      defaultProps: {
        children: `children`,
        label: `name`
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
      console.log(`z菜单数据`)
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log(`成功获取到菜单数据`, data.data)
        this.menus = data.data
      })
    },
    append (data) {
      console.log(`append`, data)
    },
    remove (node, data) {
      var ids = [data.catId]

      this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: `post`,
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            message: '菜单删除成功',
            type: 'success'
          })
          //刷新出新的菜单
          this.getMenus()
          //设置需要默认展开的菜单
          this.expandedKey = [nodethis.$parent.data.name]
        })
        console.log(`remove`, node, data)
      }).catch(() => {

      })
    }
  },
  created () {
    this.getMenus()
  }
}
</script> 

<!-- Add "scoped" attribute to limit CSS to this component only -->
<!--使用了scoped属性之后，父组件的style样式将不会渗透到子组件中，-->
<!--然而子组件的根节点元素会同时被设置了scoped的父css样式和设置了scoped的子css样式影响，-->
<!--这么设计的目的是父组件可以对子组件根元素进行布局。-->
<style scoped>
</style>
