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
  <div>
    <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId" :default-expanded-keys="expandedKey"
      draggable :allow-drop="allowDrop" @node-drop="handleDrop ">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span> {{ node.label }}</span>
        <span>
          <el-button v-if="node.level <=2" type="text" size="mini" @click="() => append(data)"> Append </el-button>

          <el-button type="text" size="mini" @click="() =>  edit(data)"> Edit </el-button>

          <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)"> Delete </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :before-close="handleClose" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.product_unit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData ">确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
export default {
  //  组件名称
  name: `demo`,
  //  组件参数 接收来自父组件的数据
  props: {},
  //  局部注册的组件
  components: {},

  data () {
    return {
      updateNodes: [],
      maxLevel: 1,
      title: ``,
      dialogType: ``, //  edit,add
      category: {
        name: ``,
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: ``,
        productUnit: ``
      },
      dialogVisible: false,
      expandedKey: [2],
      menus: [],
      defaultProps: {
        children: `children`,
        label: `name`
      }
    }
  },
  methods: {
    handleDrop (draggingNode, dropNode, dropType, ev) {
      //  第一个节点(被拖)  第二个节点（被入）  它们之间的关系
      console.log(`🚀 ~ file: category.vue ~ line 84 ~ handleDrop ~ draggingNode, dropNode, dropType`, draggingNode, dropNode, dropType)

      let ppCid, ppSort, ppLevel, siblings

      //  当前节点最新的父节点ID，当前节点最新的顺序，当前节点的最新层级
      if (dropType === `before` || dropType === `after`) {
        ppCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        ppCid = dropNode.level
        siblings = dropNode.childNodes
        console.log(`🚀 ~ file: category.vue ~ line 99 ~ handleDrop ~ siblings`, siblings)
      }

      //  如果說拖拽只改变视图，改变的是表象，那么这里的逻辑改变的就是内部逻辑关系，最后应该还得上传至数据库
      //  2、当前拖拽节点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          //  如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level
          if (siblings[i].level != draggingNode.level) {
            //  当前节点的层级发生变化
            catLevel = siblings[i].level
            //  修改它子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i, parentCid: ppCid, catLevel: catLevel })
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i })
        }
      }

      //  3、当前拖拽节点的最新层级
      console.log(`🚀 ~ file: category.vue ~ line 105 ~ handleDrop ~ updateNodes`, this.updateNodes)
    },
    updateChildNodeLevel (node) {
      console.log(`🚀 ~ file: category.vue ~ line 124 ~ updateChildNodeLevel ~ childNodes`, node.childNodes)
      if (node.childNodes.length > 0) {
        console.log(`🚀 ~ file: category.vue ~ line 125 ~ updateChildNodeLevel ~ childNodes.length`, node.childNodes.length)

        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data
          this.updateNodes.push({ catId: cNode.catId, catLevel: node.childNodes[i].level })
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    allowDrop (draggingNode, dropNode, type) {
      // console.log(`🚀 ~ file: category.vue ~ line 87 ~ allowDrop ~ draggingNode, dropNode, type`, draggingNode, dropNode, type)
      this.countNodeLevel(draggingNode)

      //  当前正在拖动的节点+父节点所在的深度不大于3即可
      //  deep为子往下
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1

      //   this.maxLevel
      if (type === `inner`) {
        return deep + dropNode.data.catLevel <= 3
      } else {
        return deep + dropNode.parent.level <= 3
      }
    },
    countNodeLevel (node) {
      //  找到所有子节点，求出最大深度
      if (node.data.children != null && node.data.children.length > 0) {
        for (let i = 0; i < node.data.children.length; i++) {
          if (node.data.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.data.children[i].catLevel
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    submitData () {
      if (this.dialogType === `add`) {
        this.addCategory()
      }
      if (this.dialogType === `edit`) {
        this.editCategory()
      }
    },
    //  修改三级分类数据
    editCategory () {
      var { catId, name, icon, productUnit } = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: `post`,
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: '菜单修改成功',
          type: 'success'
        })
        //  关闭对话框
        this.dialogVisible = false
        //  刷新出新的菜单
        this.getMenus()
        //  设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid]
      })
    },
    edit (data) {
      console.log(`要修改的数据`, data)
      this.title = `修改分类`
      this.dialogType = `edit`
      this.dialogVisible = true
      //  发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        //  请求成功
        console.log(`要回显的数据`, data)
        this.category.name = data.category.name
        this.category.catId = data.category.catId
        this.category.icon = data.category.icon
        this.category.productUnit = data.category.productUnit
        this.category.parentCid = data.category.parentCid
        this.category.catLevel = data.category.catLevel
        this.category.sort = data.category.sort
        this.category.showStatus = data.category.showStatus
      })
    },
    handleClose (data) {
      console.log(`在关闭之前执行这个函数`)
    },
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
      this.title = `添加分类`
      this.dialogType = `add`
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.name = ``
      this.category.catId = null
      this.category.icon = ``
      this.category.productUnit = ``
      this.category.sort = 0
      this.category.showStatus = 1
    },
    //  添加三级分类数据
    addCategory () {
      console.log(`提交的三级分类数据`, this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: `post`,
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: '菜单保存成功',
          type: 'success'
        })
        //  关闭对话框
        this.dialogVisible = false
        //  刷新出新的菜单
        this.getMenus()
        //  设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid]
      })
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
          //  刷新出新的菜单
          this.getMenus()
          //  设置需要默认展开的菜单
          this.expandedKey = [this.category.parentCid]
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
