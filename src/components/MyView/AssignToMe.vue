<template>
  <div class="mt-body">
    <el-container>
      <el-main>
        <el-row>
          <el-col :span="24" class="content-title-col">
            <div :class="{'content-title-item':true, 'active':isActive}">
              <i class="el-icon-date content-title-item-icon"></i>
              <span class="content-title-item-header">{{header1}}</span>
            </div>
            <el-divider direction="vertical"></el-divider>
            <div class="content-title-item" @click="switchToPT" >
              <i class="el-icon-data-analysis content-title-item-icon"></i>
              <span class="content-title-item-header">{{header2}}</span>
            </div>
          </el-col>
        </el-row>
        <el-row class="tl-bar">
          <el-col :span="24">
            <div class="tl-bar-item">
              <div style="padding:10px;margin: -14px;">
                <el-badge :value="statusCount.draftingcount" class="item1">
                  <el-button @click.native="getTaskListByStatus('Drafting')">Drafting</el-button>
                </el-badge>
                <el-badge :value="statusCount.planningcount" class="item1">
                  <el-button @click.native="getTaskListByStatus('Planning')">Planning</el-button>
                </el-badge>
                <el-badge :value="statusCount.runningcount" class="item1" type="primary">
                  <el-button @click.native="getTaskListByStatus('Running')">Running</el-button>
                </el-badge>
                <el-badge :value="statusCount.donecount" class="item1" type="warning">
                  <el-button @click.native="getTaskListByStatus('Done')">Done</el-button>
                </el-badge>   
                  <el-button @click="getTaskList()" type="primary" class="item1">Refresh</el-button>      
              </div>
              <el-input style="margin-right:30px" placeholder="Search task..." v-model="searchVal" class="tl-bar-item-input" clearable @keyup.enter.native="searchTask">
                <el-button slot="append" icon="el-icon-search" @click="searchTask"></el-button>
              </el-input>
              <el-select
                v-model="taskGroupsVal"
                multiple
                filterable
                allow-create
                default-first-option
                placeholder="Task groups">
                <el-option
                  v-for="item in taskGroups"
                  :key="item.Id"
                  :label="item.Name"
                  :value="item.Name">
                </el-option>
              </el-select>
              <el-button class="tl-bar-item-btn" type="primary"  @click="filterTask">Filter</el-button>            
            </div>
          </el-col>
        </el-row>
        <el-row class="tl-main">
          <el-col :span="7" style="margin-top:1px; ">
            <el-card class="box-card" shadow="hover" >
              <div slot="header" class="clearfix">
                <span>Task Warning</span>
                <el-button style="float: right; padding: 3px 0" v-if="false" type="text">操作按钮</el-button>
              </div>
              <span>You've got {{taskWarning.length}} tasks not complete in this sprint!</span>
                <el-table
                  v-loading="taskWarningLoading"
                  :data="taskWarning.slice((currentPage1-1)*pageSize1,currentPage1*pageSize1)"
                  style="width: 100%">
                  <el-table-column
                    prop="task_name"
                    label="Task Name"
                    width="160">
                  </el-table-column>
                  <el-table-column
                    prop="task_degree"
                    label="Degree"
                    width="80">
                  </el-table-column>
                  <el-table-column
                    prop="task_taskGroup.EndTime"
                    label="EndTime"
                    width="180">
                  </el-table-column>
                  <el-table-column fixed="right" label="Edit" align="center" width="50px">
                    <template slot-scope="scope">
                      <el-button @click="openTaskById(scope.row.task_id)" :style="{'background-color': btnColor, 'border': 'none', 'color': 'white'}" size="small" icon="el-icon-edit"></el-button>
                      </template>
                  </el-table-column>
                </el-table>
                  <el-pagination
                    @current-change="handleCurrentChange1"
                    :current-page="currentPage1"
                    :page-sizes="[5, 10, 50, 100]"
                    :page-size="pageSize1"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="taskWarning.length">
                  </el-pagination>                             
            </el-card>
          </el-col>
          <el-col :span="17" >
            <el-table v-loading="taskslistLoading" :data="taskslistData.slice((currentPage-1)*pageSize,currentPage*pageSize)" class="tl-main-table"  fit empty-text="No Data"
              row-key="task_name" lazy 
              :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
              <el-table-column prop="task_name" label="Number" width="180px" key="1"></el-table-column>
              <el-table-column prop="task_desc" label="Title" show-overflow-tooltip align="left" min-width="230px" key="2"></el-table-column>
              <el-table-column prop="task_status" label="Status" align="center" width="130px" key="3"></el-table-column>
            </el-table>
              <el-pagination
                @current-change="handleCurrentChange"
                :current-page="currentPage"
                :page-sizes="[10, 50, 100, 500]"
                :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="taskslistData.length">
              </el-pagination>            
          </el-col>
        </el-row>
      </el-main>
    </el-container> -
  </div>
</template>

<script>
import http from '../../utils/http'
import utils from '../../utils/utils'
export default {
  name: 'AssignToMe',
  data () {
    return {
      header1: 'Assign To Me',
      header2: 'Pie Charts',
      isActive: true,
      headerColor: utils.themeStyle[this.$store.getters.getThemeStyle].headerColor,
      btnColor: utils.themeStyle[this.$store.getters.getThemeStyle].btnColor,
      assignToMeList: [],
      taskslistData: [],
      taskslistLoading:false,
      taskWarningLoading:false,
      statusCount:{
        planningcount: 0,
        draftingcount: 0,
        runningcount: 0,
        donecount: 0        
      },
      searchVal: '',
      taskWarning: [],
      taskGroups: [],
      taskGroupsVal: '',
      currentPage: 1,
      currentPage1:1,
      pageSize1:5,
      pageSize: 10   
    }
  },
  methods: {
    switchToPT () {
      this.$data.isActive = false
      console.log("PieCharts")
      this.$router.push({path: 'PieCharts'})
    },
    openTaskById (iTaskId) {
      var reqTaskId = iTaskId
      var url = '/tasks/getTaskById'
      var criteria = {
        reqTaskId: reqTaskId
      }
      this.getTask(url, criteria)
    },
    async getTask (iUrl, iCriteria) {
      const res = await http.post(iUrl, iCriteria)
      console.log(res)
      if (res.data.status === 0) {
        var rtnTask = res.data.data
        if (rtnTask.task_level === 3) {
          // Clear existing data

        }
        if (rtnTask.task_level === 4) {
          // Clear existing data

        }
      }
    },
    async filterTask () {
      console.log(this.$data.taskGroupsVal)
      for(var i = 0 ; i < this.$data.taskGroupsVal.length ; i ++){
        const res = await http.post('/tasks/getTaskGroupIdByName',{
            reqTaskGroupName:this.$data.taskGroupsVal[i]
        })
        console.log(res)        
      }
    },
    async getTaskListByStatus (iStatus) {
      console.log(iStatus)
      const res = await http.post('/tasks/getTaskListByStatus',{
        AssignId : 1,
        reqStatus : iStatus
      })
      console.log(res)
      this.$data.taskslistData = res.data.data
    },
    handleCurrentChange (val) {
      console.log("handleCurrentChange")
      this.$data.currentPage = val
      this.getTaskList()
    },
    handleCurrentChange1 (val) {
      var pageSize1 = this.$data.pageSize1
      this.$data.currentPage1 = val
      this.gettaskWarning()
    },
    getCurrentMonthFirst () {
      var date = new Date()
      date.setDate(1)
      var month = parseInt(date.getMonth() + 1)
      var day = date.getDate()
      if (month < 10) {
        month = '0' + month
      }
      if (day < 10) {
        day = '0' + day
      }
      var firstDate = new Date(date.getFullYear(), month - 1, day)
      return firstDate
    },
    async getTaskList () {
      //this.$data.taskslistLoading = true
      console.log('Start to get task list')
      const res = await http.get('/tasks/getAssignToTaskLevel3',{
        AssignId: 1,
      })
      var Parenttaskname = []
      for(var i = 0 ; i < res.data.data.length ; i++){
        Parenttaskname.push(res.data.data[i].task_name)
      }
      const res2 = await http.get('/tasks/getAssignToTaskLevel4NotLevel3',{
          ParentTaskName: Parenttaskname,
          AssignId: 1,
        })
      for(var i = 0 ; i < res.data.data.length ; i++){
        const res1 = await http.get('/tasks/getAssignToTaskLevel4ForLevl3',{
          AssignId: 1,
          ParentTaskName: res.data.data[i].task_name,
        })
        if(res1.data.status!=1||res1.data.data!=null){
           res.data.data[i].children = res1.data.data
           for(var j = 0 ;j < res1.data.data.length ; j++){
           }
        }
      }
      this.$data.taskslistData = res.data.data
      for(var i = 0 ; i < res2.data.data.length;i++){
        this.$data.taskslistData.push(res2.data.data[i])
      }
      const res3 = await http.post('/tasks/getTaskGroup')      
      console.log(res3)
      this.$data.taskGroups =  res3.data.data
      //this.$data.taskslistLoading = false      
      this.countStatus(this.$data.taskslistData)
    },
    async gettaskWarning (iTask) {
      this.$data.taskWarningLoading = true
      const res = await http.get('/tasks/getTaskWarning',{
        AssignId : 1
      }) 
      this.$data.taskWarning = res.data.data
      this.$data.taskWarningLoading = false
    },
    async searchTask () {
      const res = await http.post('/tasks/getTaskByName',{
        reqTaskName: this.$data.searchVal
      })
      this.$data.taskslistData = res.data.data
    },
    countStatus (iTaskslistData) {
      this.$data.statusCount = {
        planningcount: 0,
        draftingcount: 0,
        runningcount: 0,
        donecount: 0        
      }
      console.log(iTaskslistData)
    for(var i = 0 ; i< iTaskslistData.length ; i ++ ){
        if(iTaskslistData[i].task_status=='Planning'){
          this.$data.statusCount.planningcount++;
        }else if(iTaskslistData[i].task_status=='Drafting'){
          this.$data.statusCount.draftingcount++;
        }else if(iTaskslistData[i].task_status=='Running'){
          this.$data.statusCount.runningcount++;
        }else if(iTaskslistData[i].task_status=='Done'){
          this.$data.statusCount.donecount++;
        }
      }
    console.log(this.$data.statusCount)     
    },
    getList(){
      this.getTaskList()
      this.gettaskWarning()
    }
  },
  created () {
    var firstDate = this.getCurrentMonthFirst()
    //this.resetTimesheet(firstDate)
    this.$data.pageSize = 10
    this.$data.pageSize1 = 5
    this.$data.taskWarning = []
    this.$data.currentPage = 1
    this.$data.currentPage1 = 1
    this.$data.taskGroups = []
    this.getList()
  }
}
</script>

<style scoped>
.mt-body {
  width: 100%;
  height: auto;
}
.el-main {
  color: #333;
  text-align: center;
  height: auto;
  padding: 0;
}
.content-title-col {
  height: 35px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.content-title-item {
  height: 30px;
  width: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 10px;
  cursor: pointer;
}
.content-title-item-icon {
  height:20px;
  width: 20px;
  font-size: 20px;
}
.content-title-item-header {
  margin-left:5px;
  height:20px;
  width: auto;
  font-size: 20px;
}
/*Common Style*/
.active {
  color: #ff6348;
  border-bottom: 1px solid #ff6348;
  cursor: default;
} 
</style>
<style>
.tl-bar{
  height: 50px;
  margin-top: 20px;
  margin-left: 10px;
}

.tl-bar-item {
  height: 50px;
  width: 100%;
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
 
.tl-bar-item-btn{
  margin-left: 20px;
  border: 1px solid white;
}
.text {
  font-size: 14px;
  text-align: left;
}
.text1 {
  font-size: 14px;
}
.item1 {
  margin-bottom: 18px;
  float: left;
  margin-right: 22px;
}
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both
}
.tl-bar-item-input {
  width: 30%;
  background-color: #fff;
  margin-left: 20px;
  border-radius: 30px;
  color: black;
} 

</style>
