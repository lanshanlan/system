<template>
  <div id="makeupCouApplyDiv">
    <div class="blank">
      <div class="positionBar">
        <span>您的当前位置：</span>
        <span><a href="#/login/main/eduAdminHome" class="returnHome">首页</a></span>
        <span> > 教务审批 > 补课审批</span>
      </div>
    </div>
    <div id="tableDiv">
      <table class="operationTable">
        <thead>
          <tr>
            <td class="headTd">申请教师</td>
            <td class="headTd">课程名称</td>
            <td class="headTd">上课班级</td>
            <td class="headTd">补课时间</td>
            <td class="headTd">申请时间</td>
            <td class="headTd">申请理由</td>
            <td class="headTd">操作</td>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(application,index) in applications" :id="index">
            <td>{{ application.teacherName }}</td>
            <td>{{ application.courseName }}</td>
            <td>{{ application.className }}</td>
            <td>{{ application.lessonsChangeInfo }}</td>
            <td>{{ application.applicationTime }}</td>
            <td>{{ application.mediationReason }}</td>
            <td>
              <button class="operationButton" @click="operation(index,'true')" title="通过">√</button>
              <!--<button @click="setTrue(index)">√</button>-->
              <!--申请通过批准-->
              <button class="operationButton" @click="operation(index,'false')" title="不通过">×</button>
              <!--申请拒绝-->
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <Modal
        v-model="modal1"
        width="400"
        :mask-closable="false"
        id="modalBody"
        :styles="{top:'10rem'}">
      <div style="font-size: 1.1rem;text-align: center;">
        <p>您确定通过该申请吗?</p>
      </div>
      <div slot="footer" style="text-align: center">
        <button id="modalBtn" @click="setTrue(applications,operationIndex)">确定</button>
        <button id="modalBtn" @click="modal1 = false">取消</button>
      </div>
    </Modal>
    <Modal
        v-model="modal2"
        width="400"
        :mask-closable="false"
        id="modalBody"
        :styles="{top:'10rem'}">
      <div style="font-size: 1.1rem;text-align: center;">
        <p>您确定拒绝该申请吗?</p>
      </div>
      <div slot="footer" style="text-align: center">
        <button id="modalBtn" @click="setFalse(applications,operationIndex)">确定</button>
        <button id="modalBtn" @click="modal2 = false">取消</button>
      </div>
    </Modal>
    <Modal
        v-model="modal3"
        width="400"
        :mask-closable="false"
        id="modalBody"
        :styles="{top:'10rem'}">
      <div style="font-size: 1.1rem;text-align: center;">
        <p>{{ errorMessage }}</p>
      </div>
      <div slot="footer" style="text-align: center">
        <button id="modalBtn" @click="modal3 = false">确定</button>
      </div>
    </Modal>
  </div>
</template>

<script>
  export default {
    name: 'makeupCouApplyDiv',
    data () {
      return {
        applications: [
          /*{ applicationId:'',teacher: '张三', course: '护理学基础', className: '普高2016护理1班', courseTime: '周一上午1、2节', Type:'？？', reasonType:'？？', appTime: '2016.08.01',reason:'??' },
          { applicationId:'',teacher: '李四', course: '内科护理', className: '普高2017护理2班', courseTime: '周一上午3、4节', Type:'？？', reasonType:'？？', appTime: '2016.08.02',reason:'??' },
          { applicationId:'',teacher: '王五', course: '护理管理学', className: '普高2015护理1班', courseTime: '周二上午1、2节', Type:'？？', reasonType:'？？', appTime: '2016.08.02',reason:'??' },
          { applicationId:'',teacher: '李小红', course: '内科护理', className: '普高2016护理3班', courseTime: '周五上午7节', Type:'？？', reasonType:'？？', appTime: '2016.08.02',reason:'??' }*/
        ],
//                申请信息
        operationIndex: null,
//        对话框参数传递
        modal1: false,
//        对话框显隐
        modal2: false,
        modal3: false,
        errorMessage: ""
      }
    },
    beforeMount: function() {
//    页面dom加载前获取后端数据
      this.$http.post('./makeUpLessionHandle',{},{
//      this.$http.post('../testPhp/makeupCouApply.php',{},{
        "Content-Type":"application/json"
      }).then(function(response){
        console.log("获取申请:");
        console.log(response.body);
        var data = response.body;
        this.applications = data.applicationsList;
      },function(error){
        this.$Message.error('连接失败，请重试！');
      });
    },
    methods: {
      operation: function(operationIndex,type){
//                对话框参数传递，触发对应对话框
        this.operationIndex = operationIndex;
        if(type == "true"){
          this.modal1 = true;
        }else{
          this.modal2 = true;
        }
      },
      setTrue: function(applications,index){
//        if(confirm("您确定通过该申请吗？")){
          this.$http.post('./makeUpLessionHandle/result-button',{
//          this.$http.post('../testPhp/adjustCouApplySetTrue.php',{
            "teacherId": this.applications[index].teacherId,
            "teacherName": this.applications[index].teacherName,
            "courseName": this.applications[index].courseName,
            "lessonsChangeInfo": this.applications[index].lessonsChangeInfo,
            "useClassroom": this.applications[index].useClassroom,
            "classId": this.applications[index].classId,
            "className": this.applications[index].className,
            "applicationTime": this.applications[index].applicationTime,
            "mediationReason": this.applications[index].mediationReason,
            "giveLessonsDetailsId": this.applications[index].giveLessonsDetailsId,
            "operation": "1"
          },{
            "Content-Type":"application/json"
          }).then(function(response){
            console.log("通过申请:");
            console.log(response.body);
            var data = response.body;
            if(data.result == "1") {
              applications.splice(index, 1);
            }else{
//              this.$Message.error("操作失败,请重试!");
              this.errorMessage = "操作失败,请重试!";
              this.modal3 = true;
            }
          },function(error){
            this.$Message.error('连接失败，请重试！');
          });
//        }
        this.modal1 = false;
      },
      setFalse: function(applications,index){
//        if(confirm("您确定拒绝该申请吗？")){
          this.$http.post('./makeUpLessionHandle/result-button',{
//          this.$http.post('../testPhp/adjustCouApplySetFalse.php',{
            "teacherId": this.applications[index].teacherId,
            "teacherName": this.applications[index].teacherName,
            "courseName": this.applications[index].courseName,
            "lessonsChangeInfo": this.applications[index].lessonsChangeInfo,
            "useClassroom": this.applications[index].useClassroom,
            "classId": this.applications[index].classId,
            "className": this.applications[index].className,
            "applicationTime": this.applications[index].applicationTime,
            "mediationReason": this.applications[index].mediationReason,
            "giveLessonsDetailsId": this.applications[index].giveLessonsDetailsId,
            "operation": "0"
          },{
            "Content-Type":"application/json"
          }).then(function(response){
            console.log("拒绝申请:");
            console.log(response.body);
            var data = response.body;
            if(data.result == "1") {
              applications.splice(index, 1);
            }else{
//              this.$Message.error("操作失败,请重试!");
              this.errorMessage = "操作失败,请重试!";
              this.modal3 = true;
            }
          },function(error){
            this.$Message.error('连接失败，请重试！');
          });
//        }
        this.modal2 = false;
      }
    }
  }
</script>

<style scoped>
  html{
  }
  #makeupCouApplyDiv{
    /*页面*/
    margin: 0 auto;
    background-color: #f3f3f3;
    height: 100%;
  }
  #tableDiv{
    margin: 0 5rem;
  }
  .operationButton{
    /*操作按钮*/
    outline: none;
    border: thin solid #A6A6A6;
    background-color: white;
    color: #A6A6A6;
    border-radius: 1rem;
    font-size: 1rem;
    width: 1.45rem;
  }
  .operationButton:hover{
    background-color: red;
    color: white;
    border: red;
  }
  @media screen and (max-width:1025px) {
    html {
    }
  }
</style>
