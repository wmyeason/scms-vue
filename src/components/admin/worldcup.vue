<template>
  <div>
    <div>
      <el-button type="primary" round @click="toRegister()">报名参加</el-button>
      <span>已报名参加队伍数:{{ teamNums }} </span>

      <el-button
        type="danger"
        round
        @click="cancelRegister()"
        :disabled="teamNums == 16"
        >取消报名</el-button
      >
      <el-button
        v-if="user.userType == 1"
        type="info"
        round
        @click="toReset()"
        style="float: right"
        >重置比赛</el-button
      >
    </div>
    <div class="knockout-stage">
      <!-- 用来显示A1 B2对战信息 -->
      <div v-show="this.sixteen == ''">
        <div class="round-of-16 stage">
          <h2>Round of 16</h2>
          <div class="match" v-for="(t, index) in sixteenNum" :key="index">
            <div class="team">{{ t + "1" }}</div>

            <div class="team winner">
              {{
                sixteenNum.charAt(index % 2 == 0 ? index + 1 : index - 1) + "2"
              }}
            </div>
          </div>
        </div>
      </div>

      <div v-show="this.sixteen != ''" class="round-of-16 stage">
        <h2>Round of 16</h2>
        <div class="match" v-for="t in sixteen" :key="t.id">
          <div :class="t.result == t.aid ? 'team winner' : 'team'">
            {{ t.aname }}
            <span class="score">{{ t.ascore }}</span>
          </div>
          <div>
            <span class="a"> VS </span>
          </div>
          <div :class="t.result == t.bid ? 'team winner' : 'team'">
            {{ t.bname }} <span class="score">{{ t.bscore }}</span>
          </div>
        </div>
      </div>
      <div class="quarter-finals stage">
        <h2>Quarter-finals</h2>
        <div class="match" v-for="t in quarter" :key="t.id">
          <div :class="t.result == t.aid ? 'team winner' : 'team'">
            {{ t.aname }}
            <span class="score">{{ t.ascore }}</span>
          </div>
          <div>
            <span class="a"> VS </span>
          </div>
          <div :class="t.result == t.bid ? 'team winner' : 'team'">
            {{ t.bname }} <span class="score">{{ t.bscore }}</span>
          </div>
        </div>
      </div>
      <div class="semi-finals stage">
        <h2>Semi-finals</h2>
        <div class="match">
          <div class="team winner">B2</div>
          <div class="team">F2</div>
        </div>
        <div class="match">
          <div class="team winner">A2</div>
          <div class="team">E2</div>
        </div>
      </div>
      <div class="final stage">
        <h2>Final</h2>
        <div class="match">
          <div class="team winner">B2</div>
          <div class="team">A2</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "wordlCup",
  data() {
    return {
      sixteen: [], //存放16强比赛队伍数据
      quarter: [], //存放8强比赛队伍数据
      semi: [], //存放4强比赛队伍数据
      final: [], //存放决赛比赛队伍数据
      teamNums: "",
      user: {},
      sixteenNum: "ABCDEFGH", //存放小组号
    };
  },
  mounted() {
    this.toGetTeams();
  },
  methods: {
    GetRegisterTeam() {
      //获取有报名了的队伍信息
      axios.get("/race/startRace").then((res) => {
        console.log(res.data.data);
        this.sixteen = res.data.data.sixteen;
        this.quarter = res.data.data.quarter;
        this.semi = res.data.data.semi;
        this.final = res.data.data.final;
      });
    },
    async toGetTeams() {
      //获取有多少参加了的队伍
      this.user = JSON.parse(localStorage.getItem("user"));
      axios.get("/world/getTeams").then((res) => {
        this.teamNums = res.data.data;
        if (this.teamNums == 16) {
          this.GetRegisterTeam();
        } else {
          this.teams = "";
        }
      });
    },
    async toRegister() {
      //报名
      if (this.user.team.leaderId == this.user.userId) {
        //说明是领队 可以报名
        var id = this.user.team.teamId;
        axios.put("/world/registerWorld/" + id).then((res) => {
          if (res.data.status != 200) {
            //报名不成功
            this.$message.error(res.data.msg);
          } else {
            this.$message.success("恭喜报名成功!");
            this.toGetTeams();
            if (this.teamNums === 16) {
              //报名队伍足够了   可以开始分配比赛了
              axios.get("/race/startRace").then((res) => {
                this.teams = res.data.data;
              });
            }
          }
        });
      } else {
        //报错  不是领队无法报名
        this.$message.error("您不是领队，请让领队上线报名!");
      }
    },
    cancelRegister() {
      if (this.user.team.leaderId != this.user.userId) {
        this.$message.error("您不是领队，无法取消报名，请联系领队!");
      }
      var id = this.user.team.teamId;
      //取消报名
      this.$confirm("是否确认取消报名?", "提示", {
        confirmButtonText: "确认",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          axios.get("/world/cancelRegister/" + id).then((res) => {
            if (res.data.status == 200) {
              //取消成功
              this.$message({
                type: "success",
                message: "取消成功!",
              });
              this.toGetTeams();
            } else {
              this.$message({
                type: "warning",
                message: res.data.msg,
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消",
          });
        });
    },
    toReset() {
      //管理员可以重置比赛
      this.$confirm("是否确认重置比赛?", "提示", {
        confirmButtonText: "重置",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          axios.get("/world/toReset").then((res) => {
            //TODO  处理重置比赛的后续
            console.log(res);
          });
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消重置",
          });
        });
    },
  },
};
</script>

<style lang="less" scoped>
.knockout-stage {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: flex-start;
  font-size: 18px;
  font-weight: bold;
}

.stage {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 20px 10px;
  flex-grow: 1;
}

.round-of-16 {
  flex-grow: 2;
}

.quarter-finals {
  flex-grow: 1.5;
}

.semi-finals {
  flex-grow: 1;
}

.final {
  flex-grow: 0.5;
}

.match {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  position: relative;
  margin: 10px 0;
  width: 100%;
  height: 50px;
  background-color: rgb(192, 187, 187);
  border: 1px solid #d9d9d9;
  padding: 3px;
}

.match .team {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50%;
  height: 100%;
  background-color: #eee;
  color: rgb(60, 59, 59);
}

.match .team.winner {
  background-color: #cfc;
}
.score {
  position: relative;
  right: -20px;
  color: black;
  font-weight: bold;
  font-size: 25px;
}
</style>