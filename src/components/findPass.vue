<template>
  <div class="container">
    <el-form class="login_form" ref="form">
      <el-form-item>
        <label class="la"
          >用户名:
          <el-input
            v-model="form.username"
            placeholder="请输入用户名称"
            class="inType"
          ></el-input
        ></label>
      </el-form-item>
      <el-form-item>
        <label class="la"
          >电话号码:
          <el-input
            v-model="form.phone"
            name="phone"
            placeholder="请输入电话号码"
            class="inType"
            maxlength="11"
            @input="form.phone = form.phone.replace(/[^\d]/g, '')"
          ></el-input
        ></label>
      </el-form-item>

      <Vcode :show="isShow" @success="success" @close="close" />

      <el-form-item>
        <el-button class="btn1" type="primary" @click="onSubmit">找回</el-button>
        <el-button class="btn2" @click="toLogin">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import Vcode from "vue-puzzle-vcode";
export default {
  name: "findPass",
  data() {
    return {
      form: {
        username: "",
        phone: "",
      },
      isShow: false, //验证码是否显示
    };
  },
  components: {
    Vcode,
  },
  methods: {
    onSubmit() {
      console.log("submit!");
      this.isShow = true;
    },
    // 用户通过了验证
    success(msg) {
      this.isShow = false; // 通过验证后，需要手动隐藏模态框

      //向后端发送请求   是否用户名和邮箱存在
      console.log(this.form.name);
      this.$http.post("/user/checkName", this.form).then((res) => {
        console.log(res.data.status);
        if (res.data.status !== 200) {
          console.log(res.data.msg);
          return this.$message.error(res.data.msg);
        }
      });
      //通过验证 发送请求  跳到新的设置密码页面
      this.$router.push({
        path: "/toSetNewPass",
        query: { username: this.form.username },
      });
    },
    // 用户点击遮罩层，应该关闭模态框
    close() {
      this.isShow = false;
    },
    toLogin() {
      this.$router.push("/login");
    },
  },
};
</script>

<style lang="less" scoped>
.container {
  background-image: linear-gradient(
    200deg,
    #0093e9 5%,
    #dcd2c6 60%,
    #800020 120%
  );
}
.login_form {
  width: 500px;
  height: 250px;
  background-color: #cdddd9a7;
  border-radius: 10px;

  position: absolute;

  left: 550px;
  top: 200px;

  padding: 0 10px;
  box-sizing: border-box;
}
.inType {
  width: 300px;
  height: 30px;
  border: 2px solid #ccc;
  border-radius: 5px;
  background-color: #fff;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
  float:right
}
.la {
  font-size: 30px;
  position: relative;
  color: rgb(85, 66, 104);
  top:24px;
  padding-top: 10px;
  
}
.btn1{
  position: relative;
  top:64px;
  float: right;
}
.btn2{
  position: relative;
  top:64px;
  float: left;
}
</style>