<template>
  <div class="user">
    <div class="user-info">
      <div style="line-height:35px;border-bottom:1px solid rgba(200, 200, 200, 0.6)">用户信息维护</div>
      <el-row style="width:100%;margin: 0 auto;margin-top:10px">
        <el-col :span="24">
          <div class="grid-content bg-purple-dark" style="height:50px">
            <el-form
              ref="queryForm"
              :model="queryForm"
              label-width="80px"
              class="user-queryForm"
              style="line-heigh:50px;overflow-x:hidden;margin-left:-5px"
            >
              <el-form-item v-if="userType==='2'" label="企业简称:" style="float:left;">
                <el-input v-model="queryForm.compSimpName" />
              </el-form-item>
              <el-form-item v-if="userType!=='2'" label="企业名称:" style="float:left;">
                <el-input :value="showCompName" :disabled="true" style="width:300px" />
              </el-form-item>
              <el-form-item v-if="userType==='2'" label="企业编码:" style="float:left;">
                <el-input v-model="queryForm.compCD" />
              </el-form-item>
              <el-form-item v-if="userType!='2'" label="企业编码:" style="float:left;">
                <el-input :value="showCompCd" :disabled="true" />
              </el-form-item>
              <el-form-item label="用户编码:" style="float:left;">
                <el-input v-model="queryForm.userCD" />
              </el-form-item>
              <el-form-item label="用户名称:" style="float:left;">
                <el-input v-model="queryForm.userName" />
              </el-form-item>

              <el-button
                class="user-query"
                icon="el-icon-search"
                @click="searchManagersInfo"
                style="width:118px"
              >查 询</el-button>
            </el-form>
          </div>
        </el-col>
      </el-row>

      <div class="user-infoForm">
        <el-button
          class="user-adduser"
          icon="el-icon-circle-plus"
          @click="dialogVisibleOne = true"
          v-if="userType!=0"
        >用户新增</el-button>
        <el-table
          :data="userInfoData"
          border
          :header-cell-style="{background:'rgba(48, 65, 86,0.2)'}"
          :row-style="changeRowColor"
          highlight-current-row
          v-loading="tableLoading"
          element-loading-text="数据加载中"
          element-loading-spinner="el-icon-loading"
        >
          <!-- <el-table-column label="选择" width="55" align="center">
            <template slot-scope="scope">
              <el-radio v-model="tableRadio" :label="scope.row">
                <i />
              </el-radio>
            </template>
          </el-table-column>-->

          <el-table-column label="序号" type="index" width="55" align="center" fixed="left">
            <template slot-scope="scope">
              <span>{{ (queryForm.pageNum - 1) * queryForm.pageSize + scope.$index + 1 }}</span>
            </template>
          </el-table-column>
          <el-table-column
            label="企业编码"
            align="center"
            width="150"
            prop="comp_CD"
            show-overflow-tooltip
          />
          <el-table-column label="企业简称" align="center" width="200" prop="comp_Simp_Name" />
          <el-table-column
            label="用户编码"
            align="center"
            width="150"
            prop="user_CD"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            label="用户名称"
            align="center"
            prop="user_Name"
            width="180"
            :show-overflow-tooltip="true"
          />
          <el-table-column label="用户类型" align="center" width="100">
            <template
              slot-scope="scope"
            >{{ scope.row.user_Type==1?'管理员':(scope.row.user_Type==0?"一般操作员":'超级管理员') }}</template>
          </el-table-column>
          <el-table-column label="联系电话" align="center" width="120" prop="user_Phone" />
          <el-table-column
            label="邮箱"
            align="center"
            prop="user_Email"
            :show-overflow-tooltip="true"
            width="180"
          />
          <el-table-column label="公钥地址" align="center" :show-overflow-tooltip="true" width="400">
            <template
              slot-scope="scope"
            >{{ scope.row.public_Key ==null ?'无公钥地址':scope.row.public_Key }}</template>
          </el-table-column>
          <el-table-column
            label="操作"
            align="center"
            width="280"
            v-if="userType==2||userType==1"
            fixed="right"
          >
            <template slot-scope="scope">
              <el-button size="mini" type="success" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
              <el-button
                size="mini"
                style="border:1px solid rgb(240, 182, 147);color:rgb(240, 182, 147)"
                @click="handleReset(scope.$index, scope.row)"
              >密码重置</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="block" style="text-align:center;margin-top:10px">
          <el-pagination
            :current-page="queryForm.pageNum"
            :page-sizes="[5, 10,20,50]"
            :page-size="queryForm.pageSize"
            layout="total,sizes, prev, pager, next"
            :total="totalNum"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
          />
        </div>
      </div>

      <el-dialog :visible.sync="dialogVisibleOne" title="用户新增" :close-on-click-modal="false">
        <el-form
          ref="addUserForm"
          v-loading="dialogVisibleOneLoading"
          element-loading-text="新增用户中"
          element-loading-spinner="el-icon-loading"
          :model="addUserForm"
          :rules="addUserFormRules"
          :inline="true"
          label-width="100px"
          style="margin-top:20px"
          class="user-add-user"
        >
          <!-- <el-form-item label="用户公司:" prop="userCompany">
            <el-input v-model="addUserForm.userCompany" />
          </el-form-item>-->
          <el-form-item v-if="userType==='2'" label="企业编码:" prop="compCD">
            <el-input v-model="addUserForm.compCD" />
          </el-form-item>
          <el-form-item label="用户编码:" prop="userCD">
            <el-input v-model="addUserForm.userCD" />
          </el-form-item>
          <el-form-item v-if="userType=='2'||userType=='1'" label="用户类型:" prop="userCD">
            <el-select v-model="addUserForm.userType" placeholder="请选择用户类型">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="用户名称:" prop="userName">
            <el-input v-model="addUserForm.userName" />
          </el-form-item>
          <!-- <el-form-item label="用户类型:" prop="userName">
            <el-input v-model="addUserForm.userName" />
          </el-form-item>-->
          <el-form-item label="联系电话:" prop="userPhone">
            <el-input v-model="addUserForm.userPhone" />
          </el-form-item>
          <el-form-item label="电子邮箱:" prop="userEmail">
            <el-input v-model="addUserForm.userEmail" />
          </el-form-item>
          <!-- <el-form-item label="联系人:" prop="contract">
            <el-input v-model="addUserForm.contract" />
          </el-form-item>
          <el-form-item label="联系人电话:" prop="contractPhone">
            <el-input v-model="addUserForm.contractPhone" />
          </el-form-item>-->
          <el-form-item style="width:100%;">
            <el-button @click="dialogVisibleOne = false">关闭</el-button>
            <el-button @click="resetFormOne('addUserForm')">重置</el-button>
            <el-button type="primary" @click="submitFormOne('addUserForm')">提交</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog
        :visible.sync="dialogVisibleTwo"
        style="width:50%;margin:0 auto;"
        title="信息修改"
        :close-on-click-modal="false"
      >
        <span>
          <el-form
            ref="editUserForm"
            :model="editUserForm"
            :rules="editUserFormRules"
            label-width="100px"
            class="user-modify"
            style="width:90%;margin:0 auto;"
        v-loading='loading'
         element-loading-spinner="el-icon-loading"

          >
            <!-- <el-form-item label="用户公司:" prop="userCompany">
              <el-input v-model="editUserForm.userCompany" />
            </el-form-item>-->
            <el-form-item label="用户编码:" prop="userCD">
              <el-input v-model="editUserForm.userCD" disabled />
            </el-form-item>
            <el-form-item label="用户名称:" prop="userName">
              <el-input v-model="editUserForm.userName" />
            </el-form-item>
            <!-- <el-form-item label="用户类型:" prop="userType">
              <el-input v-model="editUserForm.userType" />
            </el-form-item>-->
            <el-form-item label="联系电话:" prop="userPhone">
              <el-input v-model="editUserForm.userPhone" />
            </el-form-item>
            <el-form-item label="电子邮箱:" prop="userEmail">
              <el-input v-model="editUserForm.userEmail" />
            </el-form-item>
            <!-- <el-form-item label="联系人:" prop="contract">
              <el-input v-model="editUserForm.contract" />
            </el-form-item>
            <el-form-item label="联系人电话:" prop="contractPhone">
              <el-input v-model="editUserForm.contractPhone" />
            </el-form-item>-->
            <el-button @click="dialogVisibleTwo = false">关闭</el-button>
            <!-- <el-button @click="resetFormTwo('editUserForm')">重置</el-button> -->
            <el-button type="primary" @click="submitFormTwo('editUserForm')">提交</el-button>
          </el-form>
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import {
  getUserList,
  addManager,
  updateManagerMsg,
  delManager,
  searchManagers,
  // userMsgFromToken,
  resetOthersPSW
} from "../../api/api"
// import { getInfo } from '../../api/user'
import Bus from '../../utils/bus.js'
export default {
  data() {
    var validateuserPhone = (rule, value, callback) => {
      const reg = /^1[0-9]{10}$/
      if (!reg.test(value)) {
        callback(new Error("请输入有效的手机号码"))
      } else {
        callback()
      }
    }
    var validateEmail = (rule, value, callback) => {
      const reg = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if (!reg.test(value)) {
        callback(new Error("请输入有效的邮箱"))
      } else {
        callback()
      }
    }
    var validateInfo = (rule, value, callback) => {
      let reg = /^[a-zA-Z\d~\!@#\$%\^&\*\(\)_\-\+=\{\[\}\]\|\\:;\"\'\<,\>\.\?\/]+$/
      if (!reg.test(value)) {
        callback(new Error("编码不能包含空格和汉字"))
      } else {
        callback()
      }
    }
    return {
      queryForm: {
        compCD: "",
        compSimpName: "",
        userCD: "",
        userName: "",
        pageSize: 10,
        pageNum: 1
      },
      addUserFormRules: {
        userCD: [
          {
            required: true,
            message: "请输入用户编码",
            trigger: "blur"
          },
          ,
          {
            min: 1,
            max: 20,
            message: "长度在 1 到 20 个字符",
            trigger: "blur"
          },
          { validator: validateInfo, trigger: "blur" }
        ],
        compCD: [
          {
            required: true,
            message: "请输入企业编码",
            trigger: "blur"
          },
          ,
          {
            min: 1,
            max: 20,
            message: "长度在 1 到 20 个字符",
            trigger: "blur"
          }
          // { validator: validateInfo, trigger: "blur" }
        ],
        userName: [
          {
            required: true,
            message: "请输入用户名称",
            trigger: "blur"
          },
          {
            min: 1,
            max: 20,
            message: "长度在 1 到 20 个字符",
            trigger: "blur"
          }
        ],
        userPhone: [
          {
            required: true,
            message: "请输入联系电话",
            trigger: "blur"
          },
          { validator: validateuserPhone, trigger: "blur" }
        ],
        userEmail: [
          {
            required: true,
            message: "请输入邮箱",
            trigger: "blur"
          },
          { validator: validateEmail, trigger: "blur" }
        ]
      },
      editUserFormRules: {
        userName: [
          {
            required: true,
            message: "请输入用户名称",
            trigger: "blur"
          },
          {
            min: 1,
            max: 20,
            message: "长度在 1 到 20 个字符",
            trigger: "blur"
          }
        ],
        userPhone: [
          {
            required: true,
            message: "请输入联系电话",
            trigger: "blur"
          },
          { validator: validateuserPhone, trigger: "blur" }
        ],
        userEmail: [
          {
            required: true,
            message: "请输入邮箱",
            trigger: "blur"
          },
          { validator: validateEmail, trigger: "blur" }
        ]
      },
      // tableRadio: '',
      addUserForm: {
        userCD: "",
        compCD: "",
        userName: "",
        userPhone: "",
        userEmail: "",
        userType: ""
      },
      // resetUserForm: { userPSW: '', newUserPSW: '' },
      userType: "",

      editUserForm: { userCD: "", userName: "", userPhone: "", userEmail: "" },

      multipleSelection: [],

      addUserValue: "",
      userInfoData: [],
      dialogVisibleOne: false,
      dialogVisibleTwo: false,
      loading:false,
      // dialogVisibleThree: false,
      currentPage: 1,
      totalNum: 0,
      pageSize: 5,
      dialogVisibleOneLoading: false,
      showCompCd: "",
      showCompName: "",
      options: [
        {
          value: "1",
          label: "管理员"
        },
        {
          value: "0",
          label: "一般操作员"
        }
      ],
      tableLoading: false,
      user_Name:''
    }
  },
  created() {
    this.searchManagersInfo()
    this.userType = localStorage.getItem("user_Type")
    this.showCompCd = localStorage.getItem("comp_CD")
    this.showCompName = localStorage.getItem("comp_Name")
  },

  methods: {
    handleEdit(index, row) {
      this.dialogVisibleTwo = true
      this.editUserForm.userCD = row.user_CD
    },
    handleReset(index, row) {
      this.dialogVisibleThree = true
      resetOthersPSW({ userCD: row.user_CD }, this.$store.getters.token).then(
        res => {
          if (res.code === 1) {
            this.$notify({
              title: "成功!",
              message: "该用户密码重置成功!",
              type: "success"
            })
          } else if (res.code === 2) {
            this.$notify({
              title: "无权限!",
              message: "无权限重置该角色密码!",
              type: "warning"
            })
          }
        }
      )
    },
    handleDelete(index, row) {
      this.$confirm("此操作将永久删除该用户, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.tableLoading = true
          delManager({ userCD: row.user_CD }, this.$store.getters.token).then(
            res => {
              if (res.code === 1) {
                this.$notify({
                  title: "成功!",
                  message: "删除成功!",
                  type: "success"
                })
                this.searchManagersInfo()
                this.tableLoading = false
              } else if (res.code === 2) {
                this.$notify({
                  title: "无权限!",
                  message: "无权限删除!",
                  type: "warning"
                })
                this.searchManagersInfo()
              }
            }
          )
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          })
        })
    },

    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row)
        })
      } else {
        this.$refs.multipleTable.clearSelection()
      }
    },
    changeRowColor({ row, rowIndex }) {
      if (rowIndex % 2 === 1) {
        return "background-color:rgb(230, 230, 230)"
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    addUser() {
      this.dialogVisibleOne = true
    },
    submitFormOne(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.dialogVisibleOneLoading = true

          addManager(this.addUserForm, this.$store.getters.token).then(res => {
            this.dialogVisibleOneLoading = false

            if (res.code === 1) {
              this.$notify({
                title: "成功!",
                message: "新增用户成功!",
                type: "success"
              })

              this.dialogVisibleOne = false
              this.searchManagersInfo()
            } else if (res.code === 0) {
              this.$notify({
                title: "注意!",
                message: "该用户编码已存在!",
                type: "warning"
              })
            } else if (res.code === 2) {
              this.$notify({
                title: "注意!",
                message: "企业编码不存在!",
                type: "warning"
              })
            } else if (res.code === 3) {
              this.$notify({
                title: "注意!",
                message: "无权限,请联系管理员!",
                type: "warning"
              })
            }
          })
        }
      })
    },
    resetFormOne(formName) {
      this.$refs[formName].resetFields()
    },
    submitFormTwo(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.loading=true
          updateManagerMsg(this.editUserForm, this.$store.getters.token).then(
            res => {
          this.loading=false

              if (res.code === 1) {
                this.searchManagersInfo()
                this.$notify({
                  title: "成功!",
                  message: "修改成功!",
                  type: "success"
                })
                this.searchManagersInfo()
                this.updateUserInfo()
                this.dialogVisibleTwo = false
              } else if (res.code === 2) {
                this.$notify({
                  title: "无权限!",
                  message: "无权限修改其他管理员信息!",
                  type: "warning"
                })
              }
            }
          )
        } else {
          return false
        }
      })
    },
    resetFormTwo(formName) {
      this.$refs[formName].resetFields()
    },
    handleSizeChange(val) {
      this.queryForm.pageSize = val
      this.queryForm.pageNum = 1
      this.changePageManagersInfo()
    },
    handleCurrentChange(val) {
      this.queryForm.pageNum = val
      this.changePageManagersInfo()
    },
    getUserListInfo() {
      getUserList(
        { pageSize: this.pageSize, pageNum: this.currentPage },
        this.$store.getters.token
      ).then(res => {
        this.showCompCd = res.userLinkComp[0].comp_CD
        this.showCompName = res.userLinkComp[0].comp_Simp_Name
      })
    },

    searchManagersInfo() {
      this.tableLoading = true
      this.queryForm.pageNum = 1
      searchManagers(this.queryForm, this.$store.getters.token).then(res => {
        this.tableLoading = false
        this.userInfoData = res.temp
        this.totalNum = res.tempTotal[0].total
      })
    },
    changePageManagersInfo() {
      searchManagers(this.queryForm, this.$store.getters.token).then(res => {
        this.userInfoData = res.temp
        this.totalNum = res.tempTotal[0].total
      })
    },
    async updateUserInfo(){
      await this.$store.dispatch("user/getInfo")
      this.user_Name=localStorage.getItem('user_Name')
      Bus.$emit('user_Name', this.user_Name)

    }
  }
}
</script>

<style lang="scss" scoped>
.user {
  &-info {
    width: 95%;
    margin: 0 auto;

    // line-height: 50px;
    // position: absolute;
    &-queryForm {
      width: 90%;
      margin: 0 auto;
    }
  }
  /deep/.el-dialog__header {
    padding: 10px 10px 5px;
    height: 44px;
    background: rgba(42, 37, 66, 1) !important;
  }
  /deep/.el-dialog__headerbtn {
    position: absolute;
    /* top: 20px; */
    top: 10px;
    right: 20px;
    padding: 0;
    background: 0 0;
    border: none;
    outline: 0;
    cursor: pointer;
    font-size: 18px;
  }
  /deep/.el-dialog__title {
    line-height: 24px;
    font-size: 18px;
    color: white;

    float: left;
  }
  &-query {
    width: 100px;
    background-color: rgb(48, 65, 86);
    color: white;
    float: right;
  }
  &-adduser {
    background-color: rgb(48, 65, 86);
    color: white;
    float: left;
    margin-bottom: 10px;
    width: 118px;
  }
  &-infoForm {
    padding-top: 10px;
    width: 100%;
    margin: 0 auto;
    border-top: 1px solid rgba(200, 200, 200, 0.6);
  }
  &-add-title {
    width: 100%;
    height: 40px;
    text-align: center;

    font-size: 22px;
    margin-top: -35px;
    // background-color: red;
    border-bottom: 1px solid rgba(200, 200, 200, 0.6);
  }
  &-add-user {
    text-align: center;
  }
  &-modify {
    text-align: center;
    margin-top: 20px;
  }
  .el-row {
    margin-bottom: 20px;
    &:last-child {
      margin-bottom: 0;
    }
  }
  .el-col {
    border-radius: 4px;
  }
}
.grid-content {
  border-radius: 4px;
}
.row-bg {
  padding: 10px 0;
}
</style>

<style lang="scss">
.user {
  &-add-user {
    /deep/.el-input__inner {
      width: 300px;
    }
  }
  .el-table--enable-row-hover .el-table__body tr:hover > td {
    background-color: rgba(102, 177, 255, 0.4) !important;
  }
  .el-table--striped .el-table__body tr.el-table__row--striped.current-row td,
  .el-table__body tr.current-row > td,
  .el-table__body tr.hover-row.current-row > td,
  .el-table__body tr.hover-row.el-table__row--striped.current-row > td,
  .el-table__body tr.hover-row.el-table__row--striped > td,
  .el-table__body tr.hover-row > td {
    background-color: rgba(102, 177, 255, 0.4) !important;
  }
}
</style>
