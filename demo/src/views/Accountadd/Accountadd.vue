<template>
  <div class="account-add">
    <!-- 面板组件 -->
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>添加账号</span>
      </div>
      <div class="text item">
        <!-- 添加账号表单 -->
        <el-form
          :model="accountAddForm"
          status-icon
          :rules="rules"
          ref="accountAddForm"
          label-width="100px"
          class="demo-ruleForm"
          size="mini"
        >
          <!-- 账号 -->
          <el-form-item label="账号" prop="username">
            <el-input type="text" v-model="accountAddForm.username" autocomplete="off"></el-input>
          </el-form-item>
          <!-- 密码 -->
          <el-form-item label="密码" prop="password">
            <el-input type="text" v-model="accountAddForm.password" autocomplete="off"></el-input>
          </el-form-item>
          <!-- 确认密码 -->
          <el-form-item label="确认密码" prop="checkPwd">
            <el-input type="text" v-model="accountAddForm.checkPwd" autocomplete="off"></el-input>
          </el-form-item>
          <!-- 选择用户组 -->
          <el-form-item label="选择用户组" prop="usergroup">
            <el-select v-model="accountAddForm.usergroup" placeholder="请选择用户组">
              <el-option label="普通用户" value="普通用户"></el-option>
              <el-option label="高级管理员" value="高级管理员"></el-option>
            </el-select>
          </el-form-item>
          <!-- 登录按钮&重置按钮 -->
          <el-form-item>
            <el-button type="primary" @click="submitForm('accountAddForm')">添加</el-button>
            <el-button @click="resetForm('accountAddForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
  </div>
</template>

<script>
// 引入qs库处理post请求
import qs from "qs";
export default {
  data() {
    //走定义密码的验证
    const pass = (rules, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码!"));
      } else if (value.leng < 3 || value.length > 6) {
        callback(new Error("长度在 3 - 6 位"));
      } else {
        if (this.accountAddForm.checkPwd !== "") {
          this.$refs.accountAddForm.validateField("checkPwd"); // 调用确认密码的验证（一致性验证）
        }
        callback();
      }
    };
    //自定义确认密码的验证
    const checkPass = (rules, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码!"));
      } else if (value !== this.accountAddForm.password) {
        callback(new Error("两次输入的密码不一致!"));
      } else {
        callback();
      }
    };
    return {
      //添加账号表单数据
      accountAddForm: {
        username: "",
        password: "",
        checkPwd: "",
        usergroup: "普通用户"
      },
      //验证规则
      rules: {
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          { min: 3, max: 6, message: "长度再3 - 6 位", trigger: "blur" }
        ],
        password: [{ required: true, validator: pass, trigger: "blur" }],
        checkPwd: [{ required: true, validator: checkPass, trigger: "blur" }],
        usergroup: [
          { required: true, message: "请选择用户组", trigger: "change" }
        ]
      }
    };
  },
  methods: {
    // 点击登录按钮 触发这个函数
    submitForm(formName) {
      // 获取表单组件 调用验证方法
      this.$refs[formName].validate(valid => {
        // 如果所有验证通过 valid就是true
        if (valid) {
          // alert("前端验证通过 可以提交给后端！");
          // 后续就可以把收集的账号和密码 一起发送给后端 验证用户名和密码的正确性。
          // 获取用户输入要新增的账号数据,用一个参数对象保存
          let params = {
            username: this.accountAddForm.username,
            password: this.accountAddForm.password,
            usergroup: this.accountAddForm.usergroup
          };
          //发送ajax请求 把数据发给后端
          this.axios
            .post("http://127.0.0.1:3000/users/adduser", qs.stringify(params), {
              headers: { "Content-Type": "application/x-www-form-urlencoded" }
            })
            .then(response => {
              if (response.data.rstCode === 1) {
                this.$message({
                  type: "success",
                  message: response.data.message
                });
                //跳转到账号管理
                this.$router.push("/accountManage");
              } else {
                this.$message.error(response.data.message);
              }
            });
        } else {
          // 否则就是false
          alert("前端验证失败 不能提交给后端！");
          return false;
        }
      });
    },
    // 点击重置按钮 触发这个函数
    resetForm(formName) {
      // this.$refs.loginForm.resetFields() 获取整个表单组件 调用重置方法
      this.$refs[formName].resetFields();
    }
  }
};
</script>

<style lang="less">
.account-add {
  .el-card {
    .el-card__header {
      text-align: left;
      font-size: 20px;
      font-weight: 600;
      background-color: #f1f1f1;
    }
    .el-card__body {
      .el-form {
        width: 290px;
        .el-form-item {
          margin-bottom: 24px;
        }
      }
    }
  }
}
</style>
