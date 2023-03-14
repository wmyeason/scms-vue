<template>
  <div class="container">
    <el-form class="setpassword_form" ref="form">
      <el-form-item>
        <label class="la">用户名:</label>
        <el-input v-model="form.username" class="inType" readonly></el-input>
      </el-form-item>
      <el-form-item>
        <label class="la">密码:</label>
        <el-input
          v-model="form.password"
          placeholder="请输入密码"
          class="inType"
        ></el-input>
      </el-form-item>

      <el-form-item>
        <label class="la">确认密码:</label>
        <el-input
          v-model="form.newpass"
          placeholder="请确认密码"
          class="inType"
        ></el-input>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="onSubmit">确认设置</el-button>
        <el-button @click="toLogin">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "setNewPass",
  data() {
    return {
      form: {
        username: "",
        password: "",
        newpass: "",
      },
    };
  },
  mounted() {
    this.setValue();
  },
  methods: {
    onSubmit() {
      console.log(this.form.password);
      console.log(this.form.newpass);
      if (this.form.password !== this.form.newpass) {
        return this.$message.error("两次密码不一致，请重新确认！");
      }
      if (this.form.password.length <= 4 || this.form.password.length >= 12) {
        return this.$message({
          message: "密码长度太长或太短！",
          type: "warning",
        });
      }
      //通过 密码的限制验证  发送请求  重新设置 密码
      this.$http.post("/user/reSetPass",this.form).then((res)=>{
        //设置完密码 跳转到登录页面
        this.$router.push("/login");
      })
    },
    setValue() {
      this.form.username = this.$route.query.username;
      console.log(this.form.username);
    },
    toLogin() {
      this.$router.push("/login");
    },
  },
};
</script>

<style lang="less" scoped>
.container {
  background-color: #ffffff;
  background-image: linear-gradient(
    135deg,
    #ffffff 0%,
    #6284ff 50%,
    #ff0000 100%
  );
}
.setpassword_form {
  width: 500px;
  height: 250px;
  background-color: #dbccb1;
  border-radius: 10px;

  position: absolute;

  left: 550px;
  top: 200px;

  padding: 0 10px;
  box-sizing: border-box;
}
.inType {
  width: 300px;
}
.la {
  font-size: 30px;
  color: rgb(220, 66, 19);
  bottom: 10px;
  width: 20px;
  left: 20px;
}
</style>