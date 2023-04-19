<template>
  <div class="background">
    <div v-for="(p, index) in list" :key="index" style="">
      <el-card class="card" shadow="always">
        <el-image
          :src="require('D:/A学习资料/Java/毕业设计/scms_pic/' + p.imgUrl)"
          @click.stop="handleClickItem"
          fit="cover"
          :preview-src-list="
            list.map((item) =>
              require(`D:/A学习资料/Java/毕业设计/scms_pic/` + item.imgUrl)
            )
          "
          class="image"
      /></el-card>
    </div>
    <div class="pageBtn">
      <el-pagination
        :current-page="queryInfo.currentPage"
        :page-size="queryInfo.pageSize"
        :page-sizes="[5, 10, 20, 50]"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "rewardAll",
  data() {
    return {
      currentUser: "",
      list: [
        { pic_url: require("@/assets/reward1.png"), pic_name: "a" },
        { pic_url: require("@/assets/reward1.png"), pic_name: "b" },
        { pic_url: require("@/assets/reward1.png"), pic_name: "c" },
      ],
      queryInfo: {
        currentPage: 1,
        pageSize: 10,
        query: "",
      },
      total: 0,
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
          console.log(this.list);
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
    handleSizeChange(newSize) {
      const _this = this;
      _this.queryInfo.pageSize = newSize;
      _this.getAllReward();
    },
    handleCurrentChange(newPage) {
      const _this = this;
      _this.queryInfo.currentPage = newPage;
      _this.getAllReward();
    },
  },
};
</script>

<style lang="less" scoped>
.background {
  width: 1300px;
  height: 750px;
  background-image: url("../assets/wood2.png");
  //background-size: cover; /* 使图片充满整个元素，可能需要根据具体情况调整 */
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  display: flex;
  flex-wrap: wrap;
  right: 100px;
}
.image {
  width: 280px;
  height: 330px;
  position: relative;
  left: 100px;
  flex: 1;
  margin: 70px;
  bottom: 90px;
  left: -90px;
}
.pageBtn {
  width:400px;
  height: 40px;
  position: relative;
  left: -400px;
  bottom: -620px;
  background-color: #e7798991;
  
}
.card {
  width: 280px;
  height: 330px;
  position: relative;
  margin: 70px;
  bottom: -190px;
}
</style>