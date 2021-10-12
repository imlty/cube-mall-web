<template>
<div>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      ref="menuTree"
      :data="menus"
      :props="defaultProps"
      show-checkbox
      node-key="id"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level<=2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </div>
    </el-dialog>
</div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json 文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
//import 引入的组件需要注入到对象中才能使用
components: {},
props: {},
  data() {
    return {
      dialogType: "",
      title: "",
      category: {
        name: "",
        isShow: 1,
        isMenu: 1,
        parentId: 0,
        seq: 1,
        templateId: 42,
        id: null,
        goodsNum: 0
      },
      dialogFormVisible: false,
      expandedKey: [],
      menus: [],
      defaultProps: {
        label: "name",
        children: "children"
      }
    }
},
//计算属性 类似于 data 概念
computed: {},
//监控 data 中的数据变化
watch: {},
//方法集合
methods: {
  //获取分类菜单
  getMenus(){
    //发送请求,最终请求的是 http://127.0.0.1:8888/api/product/category/list/tree
    //http://127.0.0.1:8888/api 是 index.js文件中 已经配置好了 baseUrl
    this.$http({
      url: this.$http.adornUrl('/product/category/list/tree'),
      method: 'get'
    }).then(({data}) => {
      console.log("成功获取到了菜单数据...", data);
      this.menus = data.data;
    })
  },
  append(data) {
    console.log("点击append...", data);
    //显示对话框
    this.dialogFormVisible = true;
    //设置title及类型
    this.title = "新增分类";
    this.dialogType = "append";

    //填写新增分类信息(初始化数据)
    this.category.parentId = data.id;
    this.category.id = null;
    this.category.name = "";
    this.category.isMenu = "0";
    this.category.isShow = "1";
    this.category.templateId = 0;
    this.category.seq = 0;
    this.category.goodsNum = 0;
  },
  remove(node, data) {
    console.log("删除菜单...",node, data);

    this.$confirm(`是否删除[${data.name}]菜单?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var ids = [data.id];

          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
            //重新刷新当前菜单
            this.getMenus();
            //设置需要默认展开的菜单, 设置为 要删除节点的父节点
            this.expandedKey = [node.parent.data.id];
          });

        }).catch(() => {
        });
  },
  edit(data) {
    console.log("编辑分类...", data);
    //显示对话框
    this.dialogFormVisible = true;
    //设置title及类型
    this.title = "编辑分类";
    this.dialogType = "edit";
    //回显分类信息的操作
    this.$http({
      url: this.$http.adornUrl(`/product/category/info/${data.id}`),
      method: 'post',
    }).then(({data}) => {
      this.category.parentId = data.data.parentId;
      this.category.id = data.data.id;
      this.category.name = data.data.name;
      this.category.isMenu = data.data.isMenu;
      this.category.isShow = data.data.isShow;
      this.category.templateId = data.data.templateId;
      this.category.seq = data.data.seq;
      this.category.goodsNum = data.data.goodsNum;
    });

  },
  addCategory(){
    console.log("提交三级分类数据data...", this.category);

    this.$http({
      url: this.$http.adornUrl('/product/category/save'),
      method: 'post',
      data: this.$http.adornData(this.category, false)
    }).then(({data}) => {
      this.$message({
        type: 'success',
        message: '菜单保存成功!'
      })
      //对话框关闭
      this.dialogFormVisible = false;
      //重新刷新当前菜单
      this.getMenus();
      //设置需要默认展开的菜单, 设置为 要删除节点的父节点
      this.expandedKey = [this.category.parentId];
    });
  },
  editCategory(){
    console.log("提交编辑...三级分类数据data...", this.category);

    var {id, name} = this.category; //获取分类名称和ID
    this.$http({
      url: this.$http.adornUrl('/product/category/update'),
      method: 'post',
      data: this.$http.adornData({id, name}, false)
    }).then(({data}) => {
      this.$message({
        type: 'success',
        message: '菜单修改成功!'
      })
      //对话框关闭
      this.dialogFormVisible = false;
      //重新刷新当前菜单
      this.getMenus();
      //设置需要默认展开的菜单, 设置为 要删除节点的父节点
      this.expandedKey = [this.category.parentId];
    });
  },
  submitData(){
    //添加分类
    if(this.dialogType == "append"){
      this.addCategory();
    }
    //编辑分类
    if(this.dialogType == "edit"){
      this.editCategory();
    }
  },
  batchDelete(){
    console.log("批量删除菜单...");

    //封装要删除的id
    let ids = [];
    let names = [];
    //通过eltree菜单 获取
    let checkedNodes = this.$refs.menuTree.getCheckedNodes();
    console.log("被选中的元素", checkedNodes);
    for (let index = 0; index < checkedNodes.length; index++) {
      //把选中的CheckBox的id值 添加到ids集合中
      ids.push( checkedNodes[index].id );
      names.push( checkedNodes[index].name )
    }

    this.$confirm(`是否批量删除[${names}]菜单?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            this.$message({
              type: 'success',
              message: '菜单批量删除成功!'
            })
            //重新刷新当前菜单
            this.getMenus();
          });

        }).catch(() => {
        });
  }
},
//生命周期 - 创建完成（可以访问当前 this 实例）
created() {
  this.getMenus();
},
//生命周期 - 挂载完成（可以访问 DOM 元素）
mounted() {

},
beforeCreate() {}, //生命周期 - 创建之前
beforeMount() {}, //生命周期 - 挂载之前
beforeUpdate() {}, //生命周期 - 更新之前
updated() {}, //生命周期 - 更新之后
beforeDestroy() {}, //生命周期 - 销毁之前
destroyed() {}, //生命周期 - 销毁完成
activated() {}, //如果页面有 keep-alive 缓存功能，这个函数会触发
}
</script>

<style lang='scss' scoped>
//@import url(); 引入公共 css 类
  .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  }
</style>
