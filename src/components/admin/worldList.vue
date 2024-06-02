<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>参赛成绩管理</el-breadcrumb-item>
      <el-breadcrumb-item>足球比赛成绩录入</el-breadcrumb-item>
    </el-breadcrumb>

    <!--参数运动员列表主体-->
    <el-card>
      <el-row :gutter="25">
        <div style="float: left">
          <el-col>
            <el-select
              v-model="selStatus"
              filterable
              placeholder="请选择淘汰赛程"
              @change="page(true)"
            >
              <el-option
                v-for="item in stage"
                :key="item.status"
                :label="item.progress"
                :value="item.status"
              >
              </el-option>
            </el-select>
          </el-col>
        </div>
      </el-row>
      <!--参数运动员列表 stripe隔行变色-->
      <el-table :data="raceList" border stripe>
        <!--索引列-->

        <el-table-column type="index"></el-table-column>
        <el-table-column label="主场队伍" prop="aname"
          ><template slot-scope="scope">
            <p v-if="scope.row.aname == null">待定</p>
            <p v-else-if="scope.row.aname != null">{{ scope.row.aname }}</p>
          </template></el-table-column
        >
        <el-table-column label="客场队伍" prop="bname">
          <template slot-scope="scope">
            <p v-if="scope.row.bname == null">待定</p>
            <p v-else-if="scope.row.bname != null">{{ scope.row.bname }}</p>
          </template>
        </el-table-column>

        <el-table-column label="操作" prop="state">
          <template slot-scope="scope">
            <div>
              <!--录入分数-->
              <el-button
                icon="el-icon-edit"
                size="mini"
                style="margin-left: 10px"
                type="primary"
                :disabled="
                  scope.row.result != null ||
                  scope.row.aname == null ||
                  scope.row.bname == null
                "
                @click="
                  addDialogVisible = true;
                  oneRace = scope.row;
                  oneRace.overTime='0'
                "
                >录入分数
              </el-button>
            </div>
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

      <!--录入分数区域-->
      <el-dialog
        :visible.sync="addDialogVisible"
        title="项目记分"
        width="40%"
        @close="addDialogClosed"
      >
        <el-form
          ref="addFormRef"
          :model="oneRace"
          class="demo-ruleForm"
          label-width="180px"
        >
          <el-form-item label="主队名称">
            <el-input v-model="oneRace.aname" disabled></el-input>
          </el-form-item>
          <el-form-item label="主队得分">
            <el-input v-model.trim="oneRace.ascore"></el-input>
          </el-form-item>
          <el-form-item label="客队名称">
            <el-input v-model="oneRace.bname" disabled></el-input>
          </el-form-item>

          <el-form-item label="客队得分">
            <el-input v-model.trim="oneRace.bscore"></el-input>
          </el-form-item>

          <el-form-item label="是否进入点球大战">
            <div>
              <el-radio
                v-model="oneRace.overTime"
                label="1"
                border
                size="medium"
                @change="addDomain"
                >是</el-radio
              >
              <el-radio
                v-model="oneRace.overTime"
                label="0"
                border
                size="medium"
                @change="removeDomain"
                
                >否</el-radio
              >
            </div>
          </el-form-item>
          <el-form-item
            v-for="(domain, index) in addFormRef.domains"
            :label="index == 0 ? '主队点球得分' : '客队点球得分'"
            :key="index"
          >
            <el-input v-model="domain.value"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button type="primary" @click="addScore" slot="reference"
            >确定</el-button
          >
          <el-button
            @click="
              addDialogVisible = false;
              addFormRef.domains = [];
              oneRace.ascore='';
              oneRace.bscore='';
              oneRace.overTime='0';
            "
            >取消</el-button
          >
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "worldList",
  data() {
    return {
      addFormRef: {
        domains: [{ value: "" }],
      },
      raceList: [],
      oneRace: {},
      flag: false, //是否弹出格式不对
      currentUser: "", //获取当前用户

      queryInfo: {
        currentPage: 1,
        pageSize: 10,
        query: "",
      },
      stage: [
        {
          progress: "16强比赛",
          status: 1,
        },
        {
          progress: "8强比赛",
          status: 2,
        },
        {
          progress: "4强比赛",
          status: 3,
        },
        {
          progress: "决赛",
          status: 4,
        },
      ],
      selStatus: "",
      total: 0,
      // 对话框状态
      addDialogVisible: false,

      dialogTableVisible: false,
      EditDialogVisible: false,
      dialogFormVisible: false,
    };
  },
  created() {
    this.currentUser = JSON.parse(localStorage.getItem("user"));
    this.selStatus = this.stage[0].status;
    this.addFormRef.domains = [];
    this.page();
  },
  methods: {
    async page(isSelect) {
      if (isSelect === true) {
        console.log("asd");
        this.queryInfo.currentPage = 1;
        this.queryInfo.pageSize = 10;
      }
      const _this = this;
      console.log(this.selStatus);
      axios
        .get("/race/queryRace?status=" + this.selStatus, {
          params: _this.queryInfo,
        })
        .then((res) => {
          let data = res.data.data;
          console.log(data);
          _this.raceList = data.records;
          _this.queryInfo.currentPage = data.current;
          _this.total = data.total;
          _this.queryInfo.pageSize = data.size;
        });
    },
    //添加分数
    async addScore() {
      //如果没有输入比分  则报错
      if(this.oneRace.ascore==null||this.oneRace.ascore==''||this.oneRace.bscore==null||this.oneRace.bscore==''){
        return this.$message.error("请输入比分！");
      }
      if (
        this.oneRace.ascore == this.oneRace.bscore &&
        this.oneRace.overTime == 0
      ) {
        return this.$message.error("无法设置平局   请确认是否进入加时！");
      }
      const _this = this;
      _this.scoreHandle();
      if (!this.flag) {
        return this.$message.error("比分格式有误!");
      }
      if (_this.oneRace.score !== "") {
        axios.put("/race/updateRace", _this.oneRace).then((res) => {
          if (res.data.status != 200) {
            return _this.$message.error(res.data.msg);
          }
          _this.$message.success("操作成功");
          if(this.oneRace.id==15){
            alert('已决出冠军，请通知管理员重置比赛！');
            
          }
          _this.addDialogVisible = false;
          _this.page();
        });
      }
      this.addFormRef.domains = [];
    },
    scoreHandle() {
      //分数处理   输入值的处理
      const regex = /^[0-9]|[0-9][0-9]$|^20$/;
      if (
        !regex.test(this.oneRace.ascore) ||
        !regex.test(this.oneRace.bscore)
      ) {
        return;
      }
      if (this.addFormRef.domains.length != 0) {
        if (
          !regex.test(this.addFormRef.domains[0].value) ||
          !regex.test(this.addFormRef.domains[1].value)
        ) {
          return;
        }
        if (
          this.addFormRef.domains[0].value == this.addFormRef.domains[1].value
        ) {
          //如果加时且 比分一致  那么无法分出胜负  报错
          return this.$message.error("点球无法分出胜负，请重新设置比分！");
        }
      }

      //给oneRace  这场比赛 设置赢家和比分结果
      if (this.oneRace.overTime == 0) {
        //没有加时  直接比较结果大小
        this.oneRace.result =
          this.oneRace.ascore > this.oneRace.bscore
            ? this.oneRace.aid
            : this.oneRace.bid;
      } else {
        this.oneRace.result =
          this.addFormRef.domains[0].value > this.addFormRef.domains[1].value
            ? this.oneRace.aid
            : this.oneRace.bid;
        this.oneRace.ascore += "[" + this.addFormRef.domains[0].value + "]";
        this.oneRace.bscore += "[" + this.addFormRef.domains[1].value + "]";
      }
      this.flag = true;
      
    },
    addDomain() {
      if (this.oneRace.ascore != this.oneRace.bscore) {
        return this.$message.error("不是平局  无法进入点球!");
      }
      //进入点球   记录点球比分
      this.addFormRef.domains.push(
        {
          value: "",
          key: "a_over",
        },
        {
          value: "",
          key: "b_over",
        }
      );
    },
    removeDomain() {
      this.addFormRef.domains = [];
    },

    addDialogClosed() {
      const _this = this;
      _this.$refs.addFormRef.resetFields();
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

.myInput {
  display: flex;
}
</style>