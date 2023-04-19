<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户信息管理</el-breadcrumb-item>
      <el-breadcrumb-item>报名审核</el-breadcrumb-item>
    </el-breadcrumb>

    <!--项目列表主体-->
    <el-card>
      <!--搜索区域-->
      <el-row :gutter="25">
        <el-col :span="5">
          <!--搜索添加-->
          <el-input
            v-model="queryInfo.query"
            clearable
            placeholder="请输入项目名称"
            @clear="page"
            @keyup.enter.native="page"
          >
            <!--搜索按钮-->
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="page"
            ></el-button>
          </el-input>
        </el-col>
        <div style="float: left">
          <el-col>
            <el-select
              v-model="selectSeasonId"
              filterable
              placeholder="请选择运动会"
              @change="page(true)"
            >
              <el-option
                v-for="item in allSeasonOptions"
                :key="item.seasonId"
                :label="item.seasonName"
                :value="item.seasonId"
              >
              </el-option>
            </el-select>
          </el-col>
        </div>
      </el-row>
      <!--项目列表 stripe隔行变色-->
      <el-table :data="item" border stripe>
        <!--索引列-->

        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="运动会届时"
          prop="item.season.seasonName"
        ></el-table-column>
        <el-table-column
          label="项目名称"
          prop="item.itemName"
        ></el-table-column>
        <!-- <el-table-column label="项目性别" prop="itemSex"></el-table-column> -->
        <el-table-column
          label="运动员编号"
          prop="user.userId"
        ></el-table-column>
        <el-table-column label="项目编号" prop="item.itemId"></el-table-column>

        <el-table-column
          label="项目记分员"
          prop="user.nickname"
        ></el-table-column>
        <el-table-column
          label="项目开始时间"
          prop="item.startTime"
        ></el-table-column>
        <el-table-column label="报名时间" prop="signTime"></el-table-column>

        <el-table-column label="操作" prop="state">
          <template slot-scope="scope">
            <!--通过审核-->
            <el-popconfirm
              title="是否确认审核通过？"
              @confirm="permitCheck(scope.row.athleteId, 1)"
            >
              <el-button
                icon="el-icon-check"
                size="mid"
                type="primary"
                slot="reference"
              ></el-button
            ></el-popconfirm>
            <!--审核不通过-->
            <el-popconfirm
              title="审核是否不通过？"
              @confirm="permitCheck(scope.row.athleteId, -1)"
            >
              <el-button
                type="danger"
                icon="el-icon-s-release"
                slot="reference"
              ></el-button>
            </el-popconfirm>
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
    </el-card>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "checkUser",
  data() {
    return {
      item: [],
      queryInfo: {
        currentPage: 1,
        pageSize: 10,
        query: "",
      },
      //选择的season
      selectSeasonId: "",
      //所有的届时列表
      allSeasonOptions: [],
      total: 0,
    };
  },
  methods: {
    async page(isSelect) {
      if (isSelect === true) {
        this.queryInfo.currentPage = 1;
        this.queryInfo.pageSize = 10;
      }
      const _this = this;
      axios
        .get(
          "/athlete/queryAthlete?season.seasonId=" +
            _this.selectSeasonId +
            "&queryInfo=",
          { params: _this.queryInfo }
        )
        .then((res) => {
          
          let data = res.data.data;
          _this.item = data.records;
          _this.item=_this.item.filter((i)=>i.checkStatus===0);
          console.log(_this.item);
          _this.queryInfo.currentPage = data.current;
          _this.total = data.total;
          _this.queryInfo.pageSize = data.size;
        });
        
    },
    handleSizeChange(newSize) {
      const _this = this;
      _this.queryInfo.pageSize = newSize;
      _this.page();
    },
    handleCurrentChange(newPage) {
      const _this = this;
      _this.queryInfo.currentPage = newPage;
      _this.page();
    },
    //获取运动会届时
    getSeasons() {
      //获取可用运动会届时
      axios
        .get(
          "/season/querySeason?query=&currentPage=1&pageSize=999999999&seasonStatus=1"
        )
        .then((res) => {
          let data = res.data.data.records;
          this.allSeasonOptions = data;
          if (this.allSeasonOptions != null && this.allSeasonOptions != "") {
            this.selectSeasonId = this.allSeasonOptions[0].seasonId;
          }
          this.page();
        });
    },
    async getScorers() {
      const _this = this;
      //先获取运动会届时信息   存入到下拉框中
      _this.getSeasons();
      // axios
      //   .get("/athlete/queryAthlete?query=-1&currentPage=1&pageSize=999999999")
      //   .then((res) => {
      //     console.log(res);
      //     this.item = res.data.data.records;
          
      //   });
    },
    //通过审核
    async permitCheck(id, status) {
      console.log(id);
      console.log(status);
      //发送请求 后端审核通过
      axios
        .get("/athlete/passCheck?id=" + id + "&status=" + status)
        .then((res) => {
          this.getScorers();
        });
    },
  },
  created() {
    //获取待审核的报名信息
    this.getScorers();
  },
};
</script>

<style lang="less" scoped>
.el-breadcrumb {
  margin-bottom: 15px;
  font-size: 17px;
}

.myTable {
  border-collapse: collapse;
  margin: 0 auto;
  text-align: center;
}

.myTable td,
.myTable th {
  border: 1px solid #cad9ea;
  color: #666;
  height: 40px;
}
</style>