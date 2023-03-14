<template>
  <div>
    <!--导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>献稿模块</el-breadcrumb-item>
      <el-breadcrumb-item>添加加油稿</el-breadcrumb-item>
    </el-breadcrumb>

    <!--项目列表主体-->
    <el-card>
      <!--搜索区域-->
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

      <label>请在此输入您的加油稿：</label>
      <el-input
        type="textarea"
        :rows="10"
        placeholder="请输入内容"
        v-model="textarea"
      >
      </el-input>
      <el-button type="primary" plain style="float: right" @click="addAri()"
        >提交</el-button
      >
    </el-card>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "newArticle",
  data() {
    return {
      textarea: "",
      selectSeasonId: "",
      queryInfo: {
        currentPage: 1,
        pageSize: 10,
        query: "",
      },
      //所有的届时列表
      allSeasonOptions: [],
      //向后端传递的值
      ar: {
        u_id: "",
        content: "",
        s_id: "",
        
      },
    };
  },
  methods: {
    //获取运动会届时
    getSeasons() {
      //获取所有运动会届时
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

      //获取可用运动会届时
      axios
        .get(
          "/season/querySeason?query=&currentPage=1&pageSize=999999999&seasonStatus=1"
        )
        .then((res) => {
          let data = res.data.data.records;
          _this.seasonEnableOptions = data;
        });
    },
    async page(isSelect) {
      if (isSelect === true) {
        this.queryInfo.currentPage = 1;
        this.queryInfo.pageSize = 10;
      }
      const _this = this;
      axios
        .get(
          "/item/queryItem?season.seasonId=" +
            _this.selectSeasonId +
            "&queryInfo=",
          { params: _this.queryInfo }
        )
        .then((res) => {
          let data = res.data.data;
          _this.item = data.records;
          _this.queryInfo.currentPage = data.current;
          _this.total = data.total;
          _this.queryInfo.pageSize = data.size;
        });
      //   _this.getTemplateOptions();
    },
    addAri() {
      if(this.textarea===""){
        return this.$message.error('请填写内容！');
      }
      this.ar.content = this.textarea;
      this.ar.s_id= this.selectSeasonId;
      this.ar.u_id= JSON.parse(localStorage.getItem("user")).userId;
      console.log(this.ar);
      //向后端发送请求
      axios.post("/article/addArticle", this.ar).then((res) => {
        if(res.data.status){
            this.$message.success('添加成功！');
        }
      });
      this.textarea = ""; //清空输入框的内容
    },
  },
  created() {
    this.getSeasons();
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
