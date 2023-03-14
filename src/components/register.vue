<template>
  <div class="container">
    <el-form ref="form" :model="form" class="login_form" :rules="addFormRules">
      <el-form-item>
        <label style="font-size: 30px">请输入注册信息:</label><br />
      </el-form-item>
      <el-form-item label="登录账号" prop="username">
        <el-input v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="登录密码" prop="password">
        <el-input v-model="form.password" type="password"></el-input>
      </el-form-item>
      <el-form-item label="用户姓名" prop="nickname">
        <el-input v-model="form.nickname"></el-input>
      </el-form-item>
      <el-form-item label="电话" prop="phone">
        <el-input v-model="form.phone" maxlength="11"></el-input>
      </el-form-item>
      <el-form-item label="性别" prop="userSex">
        <el-select v-model="form.userSex" filterable placeholder="请选择">
          <el-option
            v-for="item in userSex"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="所属团体" prop="team.teamId">
        <el-select v-model="form.team.teamId" filterable disabled  placeholder="请选择">
          <el-option
            v-for="item in teamList"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="用户类别" prop="userType">
        <el-select v-model="form.userType" filterable placeholder="请选择">
          <el-option
            v-for="item in userRole"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="onSubmit">注册</el-button>
        <el-button @click="toLogin">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "register",
  data() {
    var validatePhone=(rule,value,callback)=>{
        var reg=/^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\d{8}$/;
        if(!reg.test(this.form.phone)){
          callback(new Error('电话号码格式不正确!'));
        }
        callback();
        
    };
    return {
      form: {
        userId: "",
        username: "",
        password: "",
        nickname: "",
        userSex: "",
        team: {
          teamId: "2",
        },
        userType: "",
        phone: "",
      },
      userRole: [
        {
          value: 3,
          label: "运动员",
        },
        {
          value: 2,
          label: "记分员",
        },
      ],
      userSex: [
        {
          value: "男",
          label: "男",
        },
        {
          value: "女",
          label: "女",
        },
      ],

      teamList: [],

      // 表单验证
      addFormRules: {
        username: [
          //   { required: true, message: "请输入用户名", trigger: "blur" },
          {message: "请输入用户名", trigger: "blur"},
          {
            min: 4,
            max: 30,
            message: "长度在 4 到 30 个字符",
            trigger: "blur",
          },
        ],
        password: [
          {message: "请输入密码", trigger: "blur"},
          {
            min: 6,
            max: 15,
            message: "长度在 6 到 15 个字符",
            trigger: "blur",
          },
        ],
        phone:[
          { validator:validatePhone ,trigger:"blur"},//使用的是自定义的验证方式
          
        ]
      },
    };
  },
  methods: {
    onSubmit() {
        //向后端发送请求
        axios.post("/user/register", this.form).then((res) => {
            if(res.data.status!==200){
              this.$message.error('用户名或手机号已存在！');
            }else{
              this.$router.push("/login");
            }
        });
    },
    toLogin() {
      this.$router.push("/login");
    },
  },
};
</script>

<style lang="less" scoped>
.container {
  background-color: #faaca8;
  background-image: linear-gradient(19deg, #faaca8 0%, #ddd6f3 100%);
}
.login_form {
  width: 500px;
  height: 750px;
  background-color: #ffe76f;
  border-radius: 10px;

  position: absolute;

  left: 550px;
  top:  0px;

  padding: 0 10px;
  box-sizing: border-box;
}
.inType {
  width: 300px;
}
</style>