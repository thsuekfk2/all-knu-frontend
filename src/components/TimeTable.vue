<template>
  <div class="Schedule-body">
    <div class="Schedule-wrap">
      <!--<span class="Schedule-index">총 {{ total_Grades }} 학점</span>!-->
      <div class="Scheduler">
        <Schedule
          :time-ground="['09:00', this.maxTime]"
          :week-ground="['월요일', '화요일', '수요일', '목요일', '금요일']"
          :color="['#E5E5E5', '#EAEAEA', '#D6D6D6']"
          :task-detail="this.timetable"
        >
        </Schedule>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      time_list: [
        "1a",
        "1b",
        "2a",
        "2b",
        "3a",
        "3b",
        "4a",
        "4b",
        "5a",
        "5b",
        "6a",
        "6b",
        "7a",
        "7b",
        "8a",
        "8b",
        "9a",
        "9b",
        "10",
        "11",
        "12",
        "13",
        "14",
        "15",
      ],
      userState: null,
      //total_Grades: 15, //임시로 설정
      timetable: [],
      before_timetable: [],
      maxTimeTest: [],
      maxTime: "24:00",
    };
  },
  mounted() {
    const user = JSON.parse(window.localStorage.getItem("userInfo"));
    this.userState = user;
    //user 정보가 있으면 userState에 값 넣어주기
    if (user == null) {
      //로그인이 안되어 있으므로 api 호출을 하지 않고 리다이렉트
      alert("로그인을 해야합니다");
      localStorage.removeItem("userInfo");
      localStorage.removeItem("timetable");
      this.timetable = [];
    } else if (user != null) {
      axios
        .post(
          "https://all-knu-backend.accongbox.com/knu/timetable",
          this.userState.userCookies
        )
        .then((response) => {
          localStorage.setItem("timetable", JSON.stringify(response.data));
        })
        .catch((error) => {
          if (error.response.status === 403) {
            //쿠키 정보가 부정확함, api 호출 실패 리다이렉트
            alert("로그인 다시 해주세요");
            localStorage.removeItem("userInfo");
            localStorage.removeItem("timetable");
            this.$router.push("/");
          }
        });
    }
  },
  created() {
    // console.log("created");
    // console.log(JSON.parse(localStorage.getItem("timetable")));
    this.before_timetable = JSON.parse(localStorage.getItem("timetable"));
    if (this.before_timetable) {
      const Predata = this.Pretreat_data(this.before_timetable.list.data);
      const PretimeTable = [[], [], [], [], []];
      Predata.map((row) => {
        if (row.title) {
          row.detail = row.title.split(" ")[1];
          row.title = row.title.split(" ")[0];
        }
        if (row.dateStart) {
          row.dateStart = this.timeCode(row.dateStart).dateStart;
        }
        if (row.dateEnd) {
          row.dateEnd = this.timeCode(
            row.dateEnd[row.dateEnd.length - 1]
          ).dateEnd;
        }
        if (row.dateEnd) {
          this.maxTimeTest.push(parseFloat(row.dateEnd));
          this.maxTime = Math.max.apply(null, this.maxTimeTest) + 2 + ":00";
        }
        if (row.week == "월") {
          PretimeTable[0].push(row);
        }
        if (row.week == "화") {
          PretimeTable[1].push(row);
        }
        if (row.week == "수") {
          PretimeTable[2].push(row);
        }
        if (row.week == "목") {
          PretimeTable[3].push(row);
        }
        if (row.week == "금") {
          PretimeTable[4].push(row);
        }
      });
      //console.log(PretimeTable);
      this.timetable = PretimeTable;
    }
  },
  methods: {
    timeCode(time_code) {
      var start = 540;
      for (var i = 0; i < this.time_list.length; i++) {
        if (this.time_list[i] == time_code) {
          var dateStrat_h = parseInt(start / 60);
          var dateStrat_m = start % 60;
          var dateEnd_h =
            i > 18 ? parseInt((start + 50) / 60) : parseInt((start + 25) / 60);
          var dateEnd_m = i > 18 ? (start + 50) % 60 : (start + 25) % 60;
          var date = {
            // dateStart: toString(start / 60) + ":" + toString(start % 60),
            dateStart: "".concat(
              dateStrat_h > 9 ? dateStrat_h : "0" + dateStrat_h,
              ":",
              dateStrat_m > 9 ? dateStrat_m : "0" + dateStrat_m
            ),
            dateEnd: "".concat(
              dateEnd_h > 9 ? dateEnd_h : "0" + dateEnd_h,
              ":",
              dateEnd_m > 9 ? dateEnd_m : "0" + dateEnd_m
            ),
          };
          return date;
        }
        if (i > 18) start += 50;
        else {
          start += 25;
          if ((i + 1) % 3 == 0) start += 10;
        }
      }
    },
    Pretreat_data(data) {
      const hashMap = [];
      const time = [];
      data.map((row) => {
        if (row.time_day1 != null) {
          let index = hashMap.indexOf(row.time_day1);
          if (index != -1) {
            //존재
            time[index].dateEnd.push(row.time_code);
          } else {
            //존재x
            index = hashMap.push(row.time_day1) - 1;
            time[index] = {
              title: hashMap[index],
              week: "월",
              dateStart: row.time_code,
              dateEnd: [],
              detail: [],
            };
          }
        }
        if (row.time_day2 != null) {
          let index = hashMap.indexOf(row.time_day2);
          if (index != -1) {
            //존재
            time[index].dateEnd.push(row.time_code);
          } else {
            //존재x
            index = hashMap.push(row.time_day2) - 1;
            time[index] = {
              title: hashMap[index],
              week: "화",
              dateStart: row.time_code,
              dateEnd: [],
              detail: [],
            };
          }
        }
        if (row.time_day3 != null) {
          let index = hashMap.indexOf(row.time_day3);
          if (index != -1) {
            //존재
            time[index].dateEnd.push(row.time_code);
          } else {
            //존재x
            index = hashMap.push(row.time_day3) - 1;
            time[index] = {
              title: hashMap[index],
              week: "수",
              dateStart: row.time_code,
              dateEnd: [],
              detail: [],
            };
          }
        }
        if (row.time_day4 != null) {
          let index = hashMap.indexOf(row.time_day4);
          if (index != -1) {
            //존재
            time[index].dateEnd.push(row.time_code);
          } else {
            //존재x
            index = hashMap.push(row.time_day4) - 1;
            time[index] = {
              title: hashMap[index],
              week: "목",
              dateStart: row.time_code,
              dateEnd: [],
              detail: [],
            };
          }
        }
        if (row.time_day5 != null) {
          let index = hashMap.indexOf(row.time_day5);
          if (index != -1) {
            //존재
            time[index].dateEnd.push(row.time_code);
          } else {
            //존재x
            index = hashMap.push(row.time_day5) - 1;
            time[index] = {
              title: hashMap[index],
              week: "금",
              dateStart: row.time_code,
              dateEnd: [],
              detail: [],
            };
          }
        }
      });
      return time;
    },
  },
};
</script>

<style>
ul {
  list-style: none;
  padding: 0px !important;
  color: black;
}
a,
h2,
small,
h3,
p {
  color: black !important;
}
.modal-mask p {
  color: black;
  display: flex;
  justify-content: center;
}
.time-ground {
  width: 100% !important;
}
h3 {
  display: flex !important;
  margin: 0px !important;
  width: 100% !important;
  height: 50% !important;
  justify-content: center;
  align-items: center;
}
.time-ground ul li p {
  width: 100% !important;
}
.task-ground ul li {
  width: 20% !important;
}
.Schedule-header {
  padding-left: 10px;
}
.Schedule-title {
  width: 100%;
  height: 70px;
  display: flex;
  align-items: center;
  font-size: 20px;
  font-weight: 700;
}
.Schedule-body {
  padding-left: 20px;
  height: 10%;
}
.Schedule-wrap {
  padding: 10px;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}
.Scheduler {
  width: 100%;
}
.Schedule-index {
  font-size: 15px;
  font-weight: 700;
  padding-bottom: 10px;
}
@media only screen and (max-width: 600px) {
  /* 테블릿 M일 때*/
}
@media only screen and (max-width: 1024px) {
  /* 테블릿 M일 때*/
}
@media only screen and (max-width: 768px) {
  /* 테블릿S일 때 */
}
@media only screen and (max-width: 479px) {
  /* 모바일 일 때 */
  .Schedule-title {
    font-size: 17px;
  }
  .Schedule-index {
    font-size: 12px;
    font-weight: 700;
  }
}
</style>
