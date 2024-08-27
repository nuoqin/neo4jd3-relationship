<template>
  <div id="app" style="width: 100%; height: 100%;">
    <div id="neo4jd3" class="neo4jd3" @contextmenu="showContextMenu">

    </div>
    <div v-if="contextMenuVisible" class="box-menu"
      :style="{ left: contextMenuPosition.x + 'px', top: contextMenuPosition.y + 'px' }">
      <div class="add" v-show="userGraph" @click="setQueryUser">
        <i class="el-icon-coordinate" style="padding-right: 10px;"></i>查询负责人
      </div>
      <div class="add" v-show="factoryGraph" @click="setQueryFactory">
        <i class="el-icon-office-building" style="padding-right: 10px;"></i>查询机构
      </div>
      <div class="add" @click="nextGraph">
        <i class="el-icon-circle-plus-outline" style="padding-right: 10px;"></i>关联节点
      </div>
      <div class="add" @click="hideOtherGraph">
        <i class="el-icon-circle-close" style="padding-right: 10px;"></i>隐藏(其他)
      </div>
      <div class="delete" @click="hideCurrentGraph">
        <i class="el-icon-view" style="padding-right: 10px;"></i>隐藏(当前)
      </div>
    </div>
  </div>
</template>

<script>

import { Neo4jD3 } from '@/assets/d3/js/neo4jd3'
export default {
  name: "Graphs",
  data() {
    return {
      neo4jd3: null,
      data: {},
      xmid: null,
      xmfzrid: null,
      idsClick: [],
      nodeData: {},
      userGraph: false,
      factoryGraph: false,
      drawTitle: "节点信息",
      queryForm: {
          type:["factory"],
          factoryName:null,
          endType:null,
          startType:"",
          name:""
      },
      showType: "1",
      showTitle: "组合查询",
      dialogVisible: false,
      dialogData: {},
      icon: "el-icon-arrow-right",
      iconLeft: -30,
      timeProcess: [],
      mood: "show",
      contextMenuVisible: false,
      contextMenuPosition: { x: 0, y: 0 },
      leftClickData: null
    };
  },
  created() {
    this.xmid = this.getQueryParam("xmid")
    this.xmfzrid = this.getQueryParam("xmfzrid")
  },
  mounted: function () {
    window.onresize = function () {
      document.getElementById("neo4jd3").style.height = document.documentElement.clientHeight + 'px';
      document.getElementById("neo4jd3").style.width = '100%';
      document.getElementById("neo4jd3").style.overflow = 'hidden';

    };
    window.onresize();
    window.addEventListener('click', this.hideContextMenu);

    this.loadData();
  },
  methods: {
    hideOtherGraph(){
      let node = this.leftClickData
      this.neo4jd3.hideOther(node)
    },
    hideCurrentGraph(){
      let node = this.leftClickData
      this.neo4jd3.hideCurrent(node)
    },
    handleClose(done) {
      done();
    },
    reset(){
      this.queryForm= {
          type:["factory"],
          factoryName:null,
          endType:null,
          startType:"",
          name:""
      }
    },
    transformFun() {
      this.reset()
      if (this.showType == "1") {
        this.showType = "2"
        this.showTitle = "单一查询"
      } else {
        this.showType = "1"
        this.showTitle = "组合查询"
      }
    },
    submitForm() {
      this.neo4jd3.clearNodes()
      this.neo4jd3.clearRelationship()
      if (this.queryForm.name) {
        this.queryForm.startId = null
        this.queryForm.endId = null
      } else {
        this.queryForm.startId = "user-" + this.xmfzrid
        this.queryForm.endId = "project-" + this.xmid
      }
      this.loadData()
    },
    loadData() {
      executeQuery(this.queryForm).then(res => {
        this.data = res
        this.initGraph()
      })
    },
    nextGraph() {
      let node = this.leftClickData
      this.next(node)
    },
    next(node) {
      let _this = this
      if (_this.idsClick.indexOf(node.id) != -1) {
        return;
      }
      _this.idsClick.push(node.id)
      let query = {
        startId: node.id,
        type: this.queryForm.type
      }
      executeNext(query).then(res => {
        let data = res.data[0].graph
        data = _this.neo4jd3.buildD3Data(node, data);
        _this.neo4jd3.updateWithD3Data(data);
      })
    },
    initGraph() {
      let data = this.data
      let _this = this;
      this.neo4jd3 = new Neo4jD3('#neo4jd3', {
        highlight: [{
          class: 'Project',
          property: 'name',
          value: 'neo4jd3'
        }],
        icons: {},
        images: {},
        minCollision: 100,
        neo4jData: data,
        nodeRadius: 28,
        onNodeContextmenu: function (node) {
          //鼠标右键点击事件处理
        },
        onNodeClick: function (node) {
         
        },
        onRelationshipDoubleClick: function (relationship) {
          if (relationship.type != '主持') {
            _this.dialogVisible = true
            _this.dialogData = relationship.properties
            let processId = relationship.properties.id
            if (processId) {
              timeLine(processId).then(res => {
                _this.timeProcess = res.data
              })
            }
          }
        },
        zoomFit: false
      });
    },
    hideSearch() {
      if (this.icon == "el-icon-arrow-left") {
        this.icon = "el-icon-arrow-right"
        this.mood = "show"
        this.iconLeft=-30
      } else {
        this.icon = "el-icon-arrow-left"
        this.mood = "hide"
        this.iconLeft=250
      }
    },
    showContextMenu(event) {
      // 阻止默认右键菜单
      event.preventDefault();
      if (event.target.__data__) {
        let data = event.target.__data__;
        if (data.id.indexOf("user-") != -1) {
          this.userGraph = true
          this.factoryGraph = false
        } else if (data.id.indexOf("factory-") != -1) {
          this.userGraph = false
          this.factoryGraph = true
        } else {
          this.userGraph = false
          this.factoryGraph = false
        }
        this.leftClickData = data
        // 获取右键点击的位置
        this.contextMenuPosition.x = event.clientX;
        this.contextMenuPosition.y = event.clientY;
        // 显示右键菜单
        this.contextMenuVisible = true;
      }
    },
    setQueryUser() {
      this.queryForm.name = this.leftClickData.name
      this.queryForm.startType = '院内'
    },
    setQueryFactory() {
      this.queryForm.factoryName = this.leftClickData.name
      this.queryForm.endType = '机构'
    },
    hideContextMenu() {
      // 隐藏右键菜单
      this.contextMenuVisible = false;
    },
    menuItemClicked(option) {
      // 处理菜单选项点击事件
      console.log('Clicked:', option);
      this.hideContextMenu();
    },
  },
  beforeDestroy() {
    // 移除监听器，以防止内存泄漏
    window.removeEventListener('click', this.hideContextMenu);
  }

};
</script>
<style>
.neo4jd3 {
  height: 100%;
  overflow: hidden;

}

.show {
  display: block;
}

.hide {
  display: none;
}

.margin-top {
  margin: 0px 20px;
}

.text {
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.el-timeline-item__timestamp.is-top {
  font-size: 1.1rem;
}
</style>
<style rel="stylesheet/scss" lang="scss">
.box-menu {
  width: 120px;
  position: fixed;
  border-radius: 5px;
  z-index: 1000;
  background-color: #fff;
  box-shadow: 0px 0px 10px #ccc, 0px 0px 10px #ccc, 0px 0px 10px #ccc;
  padding: 10px;

  div {
    cursor: pointer;
    line-height: 30px;
    text-align: left;
    font-size: 14px;
  }

  div:hover {
    background: rgb(29, 122, 210);
    color: white;
  }

}

:deep(.el-form-item[id=name1] .el-form-item__label){ 
    width:0px!important;
}

#name1 .el-form-item__conten{
  margin-left:0px!important;
}
:deep(.el-form-item[id=name2] .el-form-item__label){ 
    width:0px!important;
}
#name2 .el-form-item__conten{
  margin-left:0px!important;
}
</style>