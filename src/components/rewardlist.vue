<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>荣誉墙</el-breadcrumb-item>
      <el-breadcrumb-item>荣誉展示</el-breadcrumb-item>
    </el-breadcrumb>

    <el-carousel
      :interval="2000"
      type="card"
      height="800px"
      v-if="list != null && list.length > 0"
    >
      <el-carousel-item v-for="(p, index) in list" :key="index">
        <el-image
          :src="require('D:/A学习资料/Java/毕业设计/scms_pic/' + p.imgUrl)"
          @click.stop="handleClickItem"
          fit="cover"
          :preview-src-list="
            list.map((item) =>
              require(`D:/A学习资料/Java/毕业设计/scms_pic/` + item.imgUrl)
            )
          "
        />
      </el-carousel-item>
    </el-carousel>

    <el-image
      v-if="list == null || list.length == 0"
      style="width: 1250px; height: 650px"
      :src="require(`../assets/emptys.png`)"
    >
    </el-image>
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

      list: [],
    };
  },
  mounted() {
    this.currentUser = JSON.parse(localStorage.getItem("user"));
    this.getAllReward();
  },
  methods: {
    open() {
      console.log("asdas");
      this.$notify({
        title: "提示",
        message: "无荣誉展示，请继续加油",
        duration: 0,
      });
    },
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
          this.total = data.total;
          this.queryInfo.currentPage = data.current;
          this.list = data.records;
          if (this.list.length <= 0) {
            this.open();
          }
          this.queryInfo.pageSize = data.size;
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
</style>
