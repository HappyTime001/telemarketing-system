<template>
  <div class="userList-container">
    
    <!-- 搜索条件 -->
    <div class="filter-container">
      <el-input @keyup.enter.native="handleFilter" style="width: 200px;" class="filter-item" placeholder="账号" v-model="listQuery.userName">
      </el-input>
      <el-select v-model="listQuery.dianxiao" placeholder="电销"  style="width: 200px;">
         <el-option v-for="item in  dianxiaoList" :key="item.value" :label="item.label" :value="item.value">
         </el-option>
      </el-select>
      <el-date-picker
            v-model="listQuery.queryDate"
            format="yyyy-MM-dd"
            :editable="dateEditable"
            type="daterange"
            align="right"
            placeholder="选择登录时间范围"
            :picker-options="pickerOptions2">
          </el-date-picker>

      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>
      
      <!-- <el-button class="filter-item" type="primary" @click="handleCreate"  icon="edit">添加</el-button>

      <el-button class="filter-item" type="primary" @click="handleDelAll"  icon="edit">批量删除</el-button> -->
    </div>
   
    <!-- 表格 -->
    <el-table ref="multipleTable" @selection-change="handleSelectionChange" :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit highlight-current-row  >
         <el-table-column type="selection" width="65">
         </el-table-column>

          <el-table-column align="center" label='序号' width="">
            <template scope="scope">
              {{scope.$index+1}}
            </template>
          </el-table-column>

          <el-table-column label="账号" width="">
            <template scope="scope">
              {{scope.row.userName}}
            </template>
          </el-table-column>

          <el-table-column label="姓名" width="">
            <template scope="scope">
              {{scope.row.nickname}}
            </template>
          </el-table-column>

          <el-table-column label="角色" width="">
            <template scope="scope">
                {{scope.row.role | roleFilterTip}}
              
            </template>
          </el-table-column>

          <el-table-column label="备注" width="">
            <template scope="scope">
              {{scope.row.remark}}
            </template>
          </el-table-column>

          <el-table-column align="center"  label="操作"  width="220">
              <template scope="scope">
                <el-button size="small" @click="handleEdit0(scope.$index, scope.row)">分配</el-button>
                <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                 
              </template>
          </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div v-show="!listLoading" class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.currPage"
        :page-sizes="[10,20,30, 50]" :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

   

    <!-- 编辑弹窗 -->
    <el-dialog title="编辑信息" :visible.sync="dialogRuleFormVisible">
          <el-form class="small-space" :model="ruleForm" :rules="rules" ref="ruleForm" label-position="left" label-width="80px" style='width: 420px; margin-left:50px;'>

            <el-form-item label="账号">
              <el-input v-model="ruleForm.userName" readonly></el-input>
            </el-form-item>

            <el-form-item label="姓名">
              <el-input v-model="ruleForm.nickname"></el-input>
            </el-form-item>

            <el-form-item label="用户权限" prop="role">
                <el-col :span="10" >
                    <el-select v-model="ruleForm.role" placeholder="用户权限"  >
                       <el-option v-for="item in  userRoles" :key="item.value" :label="item.label" :value="item.value">
                       </el-option>
                    </el-select>
                 </el-col>
            </el-form-item>

            <el-form-item label="备注">
              <el-input v-model="ruleForm.remark"></el-input>
            </el-form-item>
           
          </el-form>

          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogRuleFormVisible = false">取 消</el-button>
           
            <el-button type="primary" @click="submitForm('ruleForm')">确 定</el-button>
          </div>
    </el-dialog>

   

  </div>
</template>

<script>
import { Message } from 'element-UI';
import { global } from '@/global/global';
import { api } from '@/global/api';
import {formatDate} from '@/filters/index';
import md5 from 'blueimp-md5';

export default {
  data() {
    
    return {
        list: null,//表格list [].push({})
        total: null,
        listLoading: false,
        listQuery: {
            currPage: 1,
            pageSize: 10,
            userName: '',
            dianxiao: '',
            queryDate: ''
        },
        //新增弹窗表单可见
        dialogFormVisible: false,

        //修改弹窗表单
        ruleForm: {
          'userName': '',
          'nickname': '',
          'role': '',
          'remark': ''
        },
        //修改弹窗表单可见
        dialogRuleFormVisible: false,

        multipleSelection: [],

        
        //修改密码弹窗可见
        changepassFormVisible: false,

        dianxiaoList: [
          {'label': '全部','value': ''},
          {'label': 'lily','value': 'lily'},
          {'label': 'lucky','value': 'lucky'},
        ],
        dateEditable:false,
        pickerOptions2: {
              shortcuts: [{
                text: '最近一周',
                onClick(picker) {
                  const end = new Date();
                  const start = new Date();
                  start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                  picker.$emit('pick', [start, end]);
                }
              }, {
                text: '最近一个月',
                onClick(picker) {
                  const end = new Date();
                  const start = new Date();
                  start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                  picker.$emit('pick', [start, end]);
                }
              }, {
                text: '最近三个月',
                onClick(picker) {
                  const end = new Date();
                  const start = new Date();
                  start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
                  picker.$emit('pick', [start, end]);
                }
              }]
        },
    }
  },
  filters: {
      formatDate,
  },
  mounted() {
    let vm = this;
    vm.getList();
  },
  methods: {
    initTemp(){
      let vm = this;
      vm.temp = {
          'userName': '',
          'password': '',
          'nickname': '',
          'role': '',
          'remark': ''
      }
    },
    initPassTemp() {
      let vm = this;
      vm.passwordForm = {
        'changePassId': '',
        'newPassword':'',
        'newPassword2':''
      }
    },
    //获取列表数据
    getList() {
        let vm = this;
        vm.listLoading = true;

        let par = vm.listQuery;
        console.log('入参1：',par)
        if(par.queryDate){
            let beginDate = formatDate(par.queryDate[0]);
            let endDate = formatDate(par.queryDate[1]);
            par.beginDate = beginDate;
            par.endDate = endDate;
        }

        console.log('入参2：',par)

        global.get( api.userList, { params: vm.listQuery },function(res){
                // console.log('获取到数据-------：',JSON.stringify(res) )
                let data = res.body;
               if(data.resultCode == 0){ 
                    
                    vm.list = data.data.data;
                    console.log('列表数据：',vm.list);
                    vm.listQuery.currPage = data.data.currPage;
                    vm.listQuery.pageSize = data.data.pageSize;
                    vm.total = data.data.total;

                    vm.listLoading = false;
                    
               }else{
                    Message({
                        showClose: true,
                        message: res.body.resultMsg,
                        type: 'error'
                    });
                    vm.listLoading = false;
               }
        }, function(res){
            vm.listLoading = false;
        }, false)
    },
    //编辑
    handleEdit(index,row){
        let vm = this;
        vm.dialogRuleFormVisible = true;
        console.log('编辑的row：',index,'-----',row);
        vm.getFormData(row._id);
    },

    //根据id查询表单数据
    getFormData (id){
        var vm = this;
        global.get( api.queryUserItem, { params:{ 'id': id } }, function(res){
            console.log('-------根据id获取表单信息：',JSON.stringify(res) )
            if(res.body.resultCode == 0){
                var res = res.body.data;
                console.log('=====',res);
                vm.ruleForm = res;
                console.log('vm.ruleForm=====',vm.ruleForm);
            }else{
                Message({
                    showClose: true,
                    message: res.body.resultMsg,
                    type: 'error'
                });
            }
        },function(res){
            //失败回调
        },true)
    },

    //提交修改表单
    submitForm(formName) {
        let vm = this;
        this.$refs[formName].validate((valid) => {
            if (valid) {
                console.log('提交入参：',this.ruleForm);
                global.post( api.modifyUser, this.ruleForm, null, function(res){
                    Message({
                        showClose: true,
                        message: '提交成功，正在跳转页面……',
                        type: 'success'
                    })
                    setTimeout(()=>{
                        vm.getList();
                    },2000)
                },function(res){
                    alert('插入数据失败，接口返回的数据为：',res)
                })
            } else {
                console.log('error submit!!');
                return false;
            }
        });
        vm.dialogRuleFormVisible = false;
    },

    
    
    //复选框
    handleSelectionChange(val) {
        this.multipleSelection = val;
    },
    //操作分页
    handleSizeChange(val) {
      this.listQuery.pageSize = val;
      this.getList();
    },
    //操作分页
    handleCurrentChange(val) {
        console.log('--------',val)
        this.listQuery.currPage = val;
        this.getList();
    },
    //搜索
    handleFilter() {
        this.getList();
    },




    
  }
};
</script>

<style scoped>
  .userList-container {
    padding: 10px;
  }
  .userList-container .el-icon-information {
      position: absolute;
      right: -18px;
      top: 10px;
  }
</style>
