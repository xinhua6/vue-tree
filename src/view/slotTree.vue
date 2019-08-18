<!-- 
 @component elementUI@2.7.2 Tree 组件二次开发
 -->
<template>
	<div v-loading="isLoading" class="comp-tree" @click="hiddenMenu">
		<!-- tree -->
		<el-tree ref="SlotTree"
			:data="setTree"
			:props="defaultProps"		
			highlight-current
      @node-drop="handleDrop"
      @node-contextmenu='rihgtClick'
      @node-click="handleNodeClick"
      draggable
			:node-key="NODE_KEY">
				<div class="comp-tr-node" slot-scope="{ node, data }">
					<!-- 编辑状态 -->
					<template v-if="node.isEdit">
            <i class="el-icon-folder"></i>
						<el-input v-model="data.name" 
							autofocus
							size="mini"
							:ref="'slotTreeInput'+data[NODE_KEY]"
							@blur.stop="handleInput(node, data)"
							@keyup.enter.native="handleInput(node, data)"></el-input>
					</template>

					<!-- 非编辑状态 -->
					<template v-else>
						<!-- 名称： 新增节点增加class（is-new） -->
						<span :class="[data[NODE_KEY] < NODE_ID_START ? 'is-new' : '', 'comp-tr-node--name']">
							<i class="el-icon-folder"></i>
              {{ node.label }}
						</span>
					</template>
				</div>
			</el-tree>
      <!-- 右键产生的选项 -->
      <div v-show="menuVisible" id="menu">
        <el-menu
          id = "rightClickMenu"
          class="el-menu-vertical"
          @select="handleRightSelect">
          <el-menu-item index="1" class="menuItem">
            <i class="el-icon-edit"></i>
            <span slot="title">重命名</span>
          </el-menu-item>
          <el-menu-item index="2" class="menuItem">
            <i class="el-icon-folder-add"></i>
            <span slot="title">新建文件夹</span>
          </el-menu-item>
          <el-menu-item index="3" class="menuItem">
            <i class="el-icon-delete"></i>
            <span slot="title">删除</span>
          </el-menu-item>
          <el-menu-item index="4" class="menuItem">
            <i class="el-icon-top"></i>
            <span slot="title">升级排序</span>
          </el-menu-item>
          <el-menu-item index="5" class="menuItem">
            <i class="el-icon-bottom"></i>
            <span slot="title">降级排序</span>
          </el-menu-item>
        </el-menu>
      </div>
	</div>
</template>

<script>
import api from '@/resource/api'

export default{
	name: 'component-tree',
	data(){
		return {
			isLoading: false,// 是否加载
			setTree: api.treelist || [],// tree数据
			NODE_KEY: 'id',// id对应字段
			MAX_LEVEL: 3,// 设定最大层级
			NODE_ID_START: 0,// 新增节点id，逐次递减
			startId: null,
			defaultProps: {// 默认设置
				children: 'children',
				label: 'name'
			},
			initParam: {// 新增参数
				name: '新增节点',
				pid: 0,
				children: []
      },
      menuVisible: false,
		}
	},
	created(){
		// 初始值
		this.startId = this.NODE_ID_START
	},
	methods: {
		handleDelete(_node, _data){// 删除节点
			console.log(_node, _data)
			// 判断是否存在子节点
			if(_data.children && _data.children.length !== 0){
				this.$message.error("此文件夹里含有其他文件夹，不能删除！")
				return false;
			}else{
				// 删除操作
				let DeletOprate = () => {
					this.$nextTick(() => {
						if(this.$refs.SlotTree){
							this.$refs.SlotTree.remove(_data)
							this.$message.success("删除成功！")
						}
					})
				}

				// 二次确认
				let ConfirmFun = () => {
					this.$confirm("是否删除此节点？","提示",{
						confirmButtonText: "确认",
						cancelButtonText: "取消",
						type: "warning"
					}).then(() => {
						DeletOprate()
					}).catch(() => {})
				}

				// 判断是否新增： 新增节点直接删除，已存在的节点要二次确认
				_data[this.NODE_KEY] < this.NODE_ID_START ? DeletOprate() : ConfirmFun()
			}
		},
		handleInput(_node, _data){// 修改节点
			console.log(_node, _data)
			// 退出编辑状态
			if(_node.isEdit){
				this.$set(_node, 'isEdit', false)
			}
		},
		handleEdit(_node, _data){// 编辑节点
			console.log(_node, _data)
			// 设置编辑状态
			if(!_node.isEdit){
				this.$set(_node, 'isEdit', true)
			}

			// 输入框聚焦
			this.$nextTick(() => {
				if(this.$refs['slotTreeInput'+_data[this.NODE_KEY]]){
					this.$refs['slotTreeInput'+_data[this.NODE_KEY]].$refs.input.focus()
				}
			})
		},
		handleAdd(_node, _data){// 新增节点
			console.log(_node, _data)
			// 判断层级
			if(_node.level >= this.MAX_LEVEL){
				this.$message.warning("当前已达到"+ this.MAX_LEVEL + "级，无法新增！")
				return false;
			}

			// 参数修改
			let obj = JSON.parse(JSON.stringify(this.initParam));// copy参数
			obj.pid = _data[this.NODE_KEY];// 父id
			obj[this.NODE_KEY] = --this.startId;// 节点id：逐次递减id
			// 判断字段是否存在
			if(!_data.children){
				this.$set(_data, 'children', [])
			}
			// 新增数据
			_data.children.push(obj)

			// 展开节点
			if(!_node.expanded){
				_node.expanded = true
			}
		},
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log('tree drop: ', dropNode.label, dropType);
    },
    rihgtClick(event, object, value, element) {
      if (this.objectID !== object.id) {
        this.objectID = object.id;
        this.menuVisible = true;
        this.DATA = object;
        this.NODE = value;
      } else {
        this.menuVisible = !this.menuVisible;
      }
      document.addEventListener('click',(e)=>{
        this.menuVisible = false;
      })
      let menu = document.querySelector("#rightClickMenu");
      /* 菜单定位基于鼠标点击位置 */
      menu.style.left = event.clientX + 20 + "px";
      menu.style.top = event.clientY - 30 + "px";
      menu.style.position = "absolute"; // 为新创建的DIV指定绝对定位
      menu.style.width = 160 + "px";
    },
    handleRightSelect(key) {
      console.log(key);
      if (key == 1) {
        // this.NodeAdd(this.NODE, this.DATA);
        this.handleEdit(this.NODE, this.DATA)
        this.menuVisible = false;
      } else if (key == 2) {
        this.handleAdd(this.NODE, this.DATA)
        this.menuVisible = false;
      } else if (key == 3) {
        this.handleDelete(this.NODE, this.DATA)
        this.menuVisible = false;
      } else if(key == 4){
        console.log('4')
        this.menuVisible = false;
      }
    },
    NodeAdd(n, d){//新增节点
			console.log(n, d)
			//判断层级
			if(n.level >= 3){
				this.$message.error("最多只支持三级！")
				return false;
			}
			//新增数据
			d.children.push({
				id: ++this.maxexpandId,
				name: '新增节点',
				pid: d.id,
				children: []
			})
			//同时展开节点
			if(!n.expanded){
				n.expanded = true
			}
    },
    handleNodeClick(node,data){
      this.menuVisible = false
    },
    hiddenMenu(event) {
      document.addEventListener('click',function(){
        this.menuVisible = false
      })
    }
	}
}
</script>

<style lang="scss">
	/* common */

	// 显示按钮
	.show-btns{
		opacity: 1;
	}

	/* common end */

	.comp-tree{
		width: 100%;
		max-width: 700px;
		max-height: 80vh;
		padding: 2em;
		overflow: auto;
		// 顶部按钮
		.comp-tr-top{
			width: 100px;
			margin-bottom: 2em;
		}
		// 自定义节点
		.comp-tr-node{
			// label
			.comp-tr-node--name{
				display: inline-block;
				line-height: 40px;
				min-height: 40px;
				// 新增
				&.is-new{
					font-weight: bold;
				}
			}
			// button
			.comp-tr-node--btns{
				margin-left: 10px;
				opacity: 0;
				transition: opacity .1s;
				.el-button{
					transform: scale(0.8);// 缩小按钮
					& + .el-button{
						margin-left: -1px;
					}
				}
			}
		}
		// 高亮显示按钮
		.is-current{
			&>.el-tree-node__content{
				.comp-tr-node--btns{
					@extend .show-btns;
				}
			}
		}
		// 悬浮显示按钮
		.el-tree-node__content{
			&:hover{
				.comp-tr-node--btns{
					@extend .show-btns;
				}
			}
		}
	}
</style>