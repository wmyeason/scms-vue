<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>荣誉墙</el-breadcrumb-item>
      <el-breadcrumb-item>荣誉展示</el-breadcrumb-item>
    </el-breadcrumb>

    <el-carousel :interval="2000" type="card" height="800px">
      <el-carousel-item v-for="(p, index) in list" :key="index">
        <el-image
          :src="require('D:/A学习资料/Java/毕业设计/scms_pic/' + p.imgUrl)"
          @click.stop="handleClickItem"
          fit="cover"
          :preview-src-list="list.map(item=>require(`D:/A学习资料/Java/毕业设计/scms_pic/`+item.imgUrl))"
        />
      </el-carousel-item>
    </el-carousel>
    <div class="background"></div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "rewardlist",
  data() {
    return {
      queryInfo: {
        currentPage: 1,
        pageSize: 10,
        query: "",
      },
      total: 0,

      list: [
        { pic_url: require("@/assets/football.png"), pic_name: "a" },
        { pic_url: require("@/assets/logo.jpg"), pic_name: "b" },
        { pic_url: require("@/assets/hbut_gate.jpg"), pic_name: "c" },
      ],
    };
  },
  mounted() {
    this.currentUser = JSON.parse(localStorage.getItem("user"));
    console.log(this.currentUser);
    this.getAllReward();
  },
  methods: {
    getAllReward() {
      axios
        .get(
          "/reward/getAll/" +
            this.currentUser.userId +
            "/" +
            this.currentUser.team.teamId,
          {
            params: this.queryInfo,
          }
        )
        .then((res) => {
          let data = res.data.data;
          console.log("as");
          this.total = data.total;
          this.queryInfo.currentPage = data.current;
          this.list = data.records;
          this.queryInfo.pageSize = data.size;
          console.log(this.queryInfo);
        
        });
    },
    handleClickItem() {
      this.$nextTick(() => {
        let domImageMask = document.querySelector(".el-image-viewer__mask"); // 获取遮罩层dom
        if (!domImageMask) {
          return;
        }
        domImageMask.addEventListener("click", () => {
          // 点击遮罩层时调用关闭按钮的 click 事件
          document.querySelector(".el-image-viewer__close").click();
        });
      });
    },
  },
};
</script>
<style lang="less" scoped>
.background {
  // background-image: url('../assets/reward_back.png');
  background-size: fill; /* 使图片充满整个元素，可能需要根据具体情况调整 */
}
</style>
