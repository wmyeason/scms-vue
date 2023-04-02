<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>我的参赛项目</el-breadcrumb-item>
      <el-breadcrumb-item>我的参赛项目</el-breadcrumb-item>
    </el-breadcrumb>

    <!--项目列表主体-->
    <el-card>
      <el-row :gutter="25">
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
      <!--参数运动员列表 stripe隔行变色-->
      <el-table :data="athletelist" border stripe>
        <!--索引列-->

        <el-table-column type="index"></el-table-column>
        <el-table-column label="学号" prop="user.userNo"></el-table-column>
        <el-table-column
          label="参数运动员"
          prop="user.nickname"
        ></el-table-column>
        <el-table-column label="性别" prop="user.userSex"></el-table-column>

        <el-table-column
          label="参赛项目"
          prop="item.itemName"
        ></el-table-column>
        <el-table-column label="地点" prop="item.itemPlace"></el-table-column>
        <el-table-column
          label="开始时间"
          prop="item.startTime"
        ></el-table-column>
        <el-table-column label="结束时间" prop="item.endTime"></el-table-column>

        <el-table-column label="报名时间" prop="signTime"></el-table-column>

        <el-table-column label="审核状态" prop="checkStatus">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-view"
              style="position: relative; left: 30px"
              @click="showDialog(scope.row.checkStatus)"
              circle
            ></el-button
          ></template>
        </el-table-column>

        <el-table-column label="操作" prop="state">
          <template slot-scope="scope">
            <!--取消报名-->
            <el-button
              :disabled="scope.row.userIds != null||shouldDisable(scope.row.item.startTime)"
              size="mini"
              type="danger"
              @click="deleteAthlete(scope.row)"
              >取消报名
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
    </el-card>
    <el-dialog :visible.sync="dialogVisible" align-center>
      <el-steps :active="2" :space="1000">
        <el-step
          title="报名审核"
          description="您已提交报名，请等待审核"
          status="success"
        ></el-step>
        <el-step
          title="审核中"
          description="请等待管理员审核..."
          :status="stepTemp.id == 0 ? 'process' : 'success'"
        ></el-step>
        <el-step
          title="审核结果"
          :description="stepTemp.description"
          :status="stepTemp.status"
        ></el-step>
      </el-steps>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ItemList",
  data() {
    return {
      athletelist: [],
      scorers: [],
      itemDetail: [],
      //所有运动会届时列表
      allSeasonOptions: [],
      //选择的届时
      selectSeasonId: "",
      userId: JSON.parse(localStorage.getItem("user")).userId,

      queryInfo: {
        currentPage: 1,
        pageSize: 10,
        query: "",
      },
      dialogVisible: false, // 控制弹窗显示状态
      activeStep: 0, // 步骤条当前激活的步骤
      stepTemp: {},
      total: 0,
      // 对话框状态
      dialogTableVisible: false,
      dialogFormVisible: false,
    };
  },
  created() {
    this.getSeasons();
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
          "/athlete/queryAthlete?item.season.seasonId=" +
            _this.selectSeasonId +
            "&user.userId=" +
            this.userId +
            "&queryInfo=",
          { params: _this.queryInfo }
        )
        .then((res) => {
          let data = res.data.data;
          _this.athletelist = data.records;
          _this.queryInfo.currentPage = data.current;
          _this.total = data.total;
          _this.queryInfo.pageSize = data.size;
        });
    },

    //获取运动会届时
    async getSeasons() {
      const _this = this;
      axios
        .get("/season/querySeason?query=&currentPage=1&pageSize=999999999")
        .then((res) => {
          let data = res.data.data.records;
          data.push({
            seasonId: 0,
            seasonStatus: 0,
            seasonName: "所有运动会",
          });
          data.forEach((item, index) => {
            if (item.seasonStatus != 0) {
              _this.selectSeasonId = item.seasonId;
            }
          });
          _this.allSeasonOptions = data;
          _this.page();
        });
    },

    async deleteAthlete(athlete) {
      const athleteId=athlete.athleteId;
      const _this = this;
      const confirmResult = await _this
        .$confirm("是否确定取消报名？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        })
        .catch((err) => err);
      if (confirmResult !== "confirm") {
        return _this.$message.info("已取消操作");
      }
      axios
        .delete("/athlete/deleteAthlete?athleteId=" + athleteId)
        .then((res) => {
          if (res.data.status == 200) {
            _this.$message.success("已取消该项目");
            _this.addDialogVisible = false;
            _this.page();
          } else {
            _this.$message.error(res.data.msg);
          }
        });
    },

    showDialog(status) {
      if (status == 0) {
      } else if (status == 1) {
        this.stepTemp.description = "审核通过！";
        this.stepTemp.status = "success";
      } else if (status == -1) {
        this.stepTemp.description = "审核不通过！";
        this.stepTemp.status = "error";
      }
      this.stepTemp.id = status;
      this.dialogVisible = true;
    },
    shouldDisable(item) {
      const itemTime = new Date(item.value)
      console.log(itemTime > this.currentTime);
      return itemTime > this.currentTime;
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