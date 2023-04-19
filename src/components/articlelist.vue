<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>大会献稿</el-breadcrumb-item>
      <el-breadcrumb-item>查看加油稿</el-breadcrumb-item>
    </el-breadcrumb>

    <div class="block">
      <span class="demonstration">最新发布 加油稿</span>
      <el-carousel height="150px">
        <el-carousel-item v-for="item in carItem" :key="item.create_time">
          <span>{{ item.user.nickname }}</span>
          <el-divider direction="vertical"></el-divider>
          <span>{{ item.season.seasonName }}</span>
          <el-divider direction="vertical"></el-divider>
          <span>{{
            item.create_time
              .toLocaleString()
              .replace(/T/g, " ")
              .replace(/\.[\d]{3}Z/, "")
          }}</span>
          <el-tooltip
            class="item"
            effect="dark"
            :content="item.content"
            placement="top-start"
          >
            <p class="con">{{ item.content }}</p>
          </el-tooltip>
        </el-carousel-item>
      </el-carousel>
    </div>

    <!--项目列表 stripe隔行变色-->
    <el-table :data="itemList" border stripe>
      <!--索引列-->

      <el-table-column type="index"></el-table-column>
      <el-table-column label="发布者" prop="user.nickname"></el-table-column>
      <el-table-column
        label="运动会"
        prop="season.seasonName"
      ></el-table-column>

      <el-table-column label="发表内容" prop="content"></el-table-column>
      <el-table-column label="发表时间" prop="create_time">
        <template slot-scope="scope">
          {{
            scope.row.create_time
              .toLocaleString()
              .replace(/T/g, " ")
              .replace(/\.[\d]{3}Z/, "")
          }}
        </template>
      </el-table-column>

      <el-table-column label="操作" prop="id" v-if="user.userId == 1">
        <template slot-scope="scope">
          <!--详情-->

          <el-button
            icon="el-icon-tickets"
            size="mini"
            type="primary"
            @click="delArticle(scope.row.id)"
            >删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <div>
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
  name: "articlelist",
  data() {
    return {
      queryInfo: {
        currentPage: 1,
        pageSize: 5,
        query: "",
      },
      itemList: [],
      total: 0,
      carItem: [],
      user: {},
    };
  },
  created() {
    this.getCarsouelAr(); //获取轮播的加油稿
    this.page(); //获取分页了的加油稿
    this.user = JSON.parse(localStorage.getItem("user"));
    console.log(this.user);
  },
  methods: {
    async page() {
      //获取分页了的加油稿
      axios
        .get("/article/queryArticle?" + "queryInfo=", {
          params: this.queryInfo,
        })
        .then((res) => {
          let data = res.data.data;
          this.itemList = data.records;
          this.queryInfo.currentPage = data.current;
          this.total = data.total;
          this.queryInfo.pageSize = data.size;
        });
    },
    getCarsouelAr() {
      //获取轮播的加油稿
      axios.get("/article/getCarAr").then((res) => {
        this.carItem = res.data.data;
      });
    },
    handleSizeChange(newSize) {
      //更换页面容量时
      const _this = this;
      _this.queryInfo.pageSize = newSize;
      _this.page();
    },
    handleCurrentChange(newPage) {
      //更换页码时
      const _this = this;
      _this.queryInfo.currentPage = newPage;
      _this.page();
    },
    async delArticle(id) {
      //管理员删除加油稿
      //发送删除请求
      const _this = this;
      const confirmResult = await _this
        .$confirm("此操作将永久删除此加油稿，是否继续？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        })
        .catch((err) => err);
      if (confirmResult !== "confirm") {
        return _this.$message.info("已取消删除");
      }
      axios.delete("/article/deleteArticle?id=" + id).then((res) => {
        if (res.status == 200) {
          _this.$message.success("删除成功");

          _this.page();
        } else {
          _this.$message.error(res.data.msg);
        }
      });
    },
  },
};
</script>

<style lang="less" scoped>
.el-carousel__item h3 {
  color: #ba2f7b;
  font-size: 24px;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
  text-align: center;
}

.el-carousel__item:nth-child(2n) {
  background-color: #51c4d3;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #93d5dc;
}
.con {
  text-align: center;
  color: #f15161;
}
</style>