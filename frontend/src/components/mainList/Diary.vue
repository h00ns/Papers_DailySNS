<template>
  <div>
    <div @click="dialog = true" id="Plus_Item">
      <div id="Plus_Img">
        <v-icon style="font-size: 5em; color: #ffb319">add</v-icon>
      </div>
      <div id="plus-under">
        <span id="Plus_Name">일기장 만들기</span>
      </div>
    </div>
    <div
      v-for="diary in viewList"
      :key="diary.id"
      @click="goDiary(diary)"
      class="Diary_Item" 
    >
      <div class="Diary_Img" :class="{ 'highlight' : highlightDiaryId === diary.id }">
        <img :src="diary.diaryCover.coverUrl" />
      </div>
      <div class="diary-under">
        <span class="Diary_Name">{{ diary.diaryTitle }}</span>
        <span class="Diary_Day">{{ diary.diaryCreatedDate }}</span>
      </div>
    </div>
    <!-- 일기장 페이지네이션 -->
    <div id="diary-pagination">
      <v-pagination
        style="margin-bottom:30px;"
        v-model="page"
        :length="Math.ceil(diaryList.length/5)"
        @input="change"
        circle
        color="#FFB300"
      ></v-pagination>
    </div>
    <!-- Dialog -->
    <v-dialog v-model="dialog" persistent max-width="443">
      <v-card id="Dialog">
        <div id="Dialog_Header">
          <v-icon
            @click="dialog = false"
            id="Dialog_Close"
            style="font-size: 2em"
            >close</v-icon
          >
        </div>
        <div id="Dialog_Content">
          <div class="Dialog_Header" style="margin-top: 20px">타이틀</div>
          <div class="Invite_Input">
            <v-text-field
              v-model="diaryTitle"
              style="margin-top: 5px"
              color="#FFB319"
              label="일기장 타이틀을 입력해주세요."
              counter="14"
              maxlength="14"
            ></v-text-field>
          </div>
          <div class="Dialog_Header" style="margin-top: 5px">멤버 초대</div>
          <div class="Invite_Input">
            <v-text-field
              class="Search_Input"
              label="아이디 검색"
              v-model="search"
              color="orange"
              v-on:keyup="memberSearch"
            ></v-text-field>
            <v-btn id="Search_Btn" icon>
              <v-icon @click="memberSearch" style="font-size: 2.3em">search</v-icon>
            </v-btn>
          </div>
          <div id="Search_List">
            <div v-if="memberList.length === 0" id="Invite_txt">검색결과가 없습니다.</div>
            <div v-for="(member,idx) in memberList" :key="member.userId" class="Search_Item">
              <div class="Search_Img">
                <img class="Img" :src="member.userProfile" />
              </div>
              <span class="Search_Name">{{member.userNickname}} </span>
              <span calss="Search_UserId">({{ member.userId }})</span>
              
              <!-- <div class="Search_Check"></div> -->
              <div class="Search_Check">
                <v-checkbox
                  class="check"
                  v-model="selected[idx]"
                  :key="member.userId"
                  :value="member.userId"
                  color="orange"
                ></v-checkbox>
              </div>
            </div>
          </div>
          <div id="Dialog_Btn_Box">
            <v-btn
              @click="create"
              style="background: #ffb319; color: white"
              class="Dialog_Btn"
              >확인</v-btn
            >
            <v-btn
              @click="dialog = false"
              style="background: #9f9f9f; color: white"
              class="Dialog_Btn"
              >취소</v-btn
            >
          </div>
        </div>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import Swal from "sweetalert2";

export default {
  data() {
    return {
      page:1,
      dialog: false,
      diaryTitle: "",
      diaryList: [],
      viewList:[],
      search: "",
      memberList: [],
      selected: [],
      currentCreateDiaryId: 0,
      highlightDiaryId: 0,
    };
  },
  methods: {
    change(num) {
      var temp = 0
      for(let i=1;i<this.diaryList.length;i++){
        if(i==num){
          this.viewList = []
          for(let i=temp;i<temp+5;i++){
            if(this.diaryList.length==i)
              break
            this.viewList.push(this.diaryList[i])
          }
        }
        temp+=5
      }
    },
    goDiary(diary) {
      this.$store.commit('setCurrentDiary', diary) // mutaion 호출 ('뮤테이션 이름, 매개변수)
      
      this.$store.dispatch('getDiaryContent', diary.id)
        .then((res) => {
          this.$store.commit('setNoteContent', res.data.note[res.data.note.length-1])
          this.$router.push("/diary").catch(() => {});
        })
    },
    create() {
      const diary = {
        coverId: 1,
        diaryTitle: this.diaryTitle,
      };
      this.$store.dispatch("diaryCreate", diary).then((response) => { // 다이어리 생성
        this.currentCreateDiaryId = response.data.id;
        this.$store.dispatch("diaryGet").then((res) => { // 다이어리 가져오기
          this.diaryList = res.data.reverse();          
          let inviteAlarmPushUser = []
          this.selected.forEach(function (item) {
            if (item != null)
              inviteAlarmPushUser.push(item)
          })
          let share = {
          'diaryId': this.currentCreateDiaryId,
          'inviteList': inviteAlarmPushUser
          }
          this.$store.dispatch("shareDiary", share).then(() => { // 다이어리 공유 요청 보내기
            this.$store.dispatch("diaryGet").then((res) => {
              this.diaryList = res.data.reverse()
              this.dialog = false;
              this.diaryTitle = "";
              this.page = 1
              this.viewList = []
              for(let i=0;i<5;i++){
                if(this.diaryList.length==i) 
                  break
                this.viewList.push(this.diaryList[i])
              }
              Swal.fire({
                icon: "success",
                title:
                  '<span style="font-size:25px;">일기장이 만들어졌습니다.</span>',
                confirmButtonColor: "#b0da9b",
                confirmButtonText: '<span style="font-size:18px;">확인</span>',
              });
            })
          });
        });
      });
    },
    memberSearch() {
      this.$store.dispatch('memberSearch', this.search)
        .then((res) => {
          this.memberList = res.data

          if(this.search == '') {
            this.memberList = []
          }
        })
    }
  },
  created() {
    this.$store.dispatch("diaryGet").then((res) => {
      console.log(res.data, 'zzzzzz')
      this.diaryList = res.data.reverse();
      for(let i=0;i<5;i++){
        if(this.diaryList.length==i) 
          break
        this.viewList.push(this.diaryList[i])
      }
      console.log(this.viewList[0].diaryCover.coverUrl, '뷰리스트')
    });
  },
  mounted() {
    const diaryIdQuery = this.$route.query.diaryId
    if (diaryIdQuery) {
      this.highlightDiaryId = diaryIdQuery
    }
  },
};
</script>

<style lang="scss" scoped>
#Plus_Item {
  display: inline-block;
  width: 286px;
  height: 432px;
  margin-bottom: 31px;
  margin-left: 31px;
}
#Plus_Img {
  width: 286px;
  height: 394px;
  box-shadow: 3px 3px 11px rgba(166, 166, 168, 0.25);  
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
}
#plus-under{
  margin-top:12px;
  display:flex;
  align-items: center;
}
#Plus_Name {
  font-size: 15px;
}
.Diary_Item {
  display: inline-block;
  /* border:1px solid red; */
  width: 286px;
  height: 432px;
  margin-bottom: 31px;
  margin-left: 31px;
}
.Diary_Img {
  width: 286px;
  height: 394px;
  /* background: peachpuff; */
  box-shadow: 3px 3px 11px rgba(166, 166, 168, 0.35);
  cursor: pointer;
  img {
    width: 100%;
    height: 100%;
  }
}
.Diary_Name {
  max-width:190px;
  font-size: 15px;
  cursor: pointer;
}
.Diary_Name:hover {
  color: #979797;
}
.Diary_Day{
  font-size: 15px;
  color:#929292;
}
.diary-under{
  margin-top:12px;
  display: flex;
  justify-content: space-between;
  align-items:center;
}
#Dialog {
  height: 524px;
}
#Dialog_Header {
  height: 36px;
  border-bottom: 1px solid #d7d7d7;
}
#Dialog_Close {
  float: right;
  line-height: 36px;
  cursor: pointer;
  margin-right: 7px;
}
#Dialog_Content {
  overflow: hidden;
  height: 488px;
  padding: 0 27px;
}
.Dialog_Header {
  font-size: 18px;
  font-weight: 600;
}
.Invite_Input {
  margin-top: 0px;
}
.Search_Input {
  width: 89%;
  display: inline-block;
}
#Search_Btn {
  // background: #ffb319;
  color: #585858;
  width: 40px;
  height: 40px;
  position: relative;
  top: -2px;
}
#Search_List {
  height: 194px;
  border: 1px solid #d7d7d7;
  padding: 14px;
}
#Dialog_Btn_Box {
  width: 164px;
  height: 32px;
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  margin-top: 15px;
}
.Dialog_Btn {
  width: 76px;
  height: 32px;
  box-shadow: none;
}
.Search_Item {
  height: 50px;
  line-height: 50px;
}
.Search_Img {
  float: left;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.Img {
  width: 43px;
  height: 43px;
  display: inline-block;
}
.Search_Name {
  margin-left: 16px;
  font-size: 16px;
}
.Search_UserId {
  color: red;
}
.Search_Check {
  height:50px;
  float: right;
}
.check{
  position:relative;
  top:-7px;
}
#Search_List {
  overflow: auto;
  -ms-overflow-style: none;
  scrollbar-width: none;
}
#Search_List::-webkit-scrollbar {
  display: none;
}
#Invite_txt {
  line-height: 166px;
  text-align: center;
  font-size: 16px;
  color: #9f9f9f;
}
.highlight {
  background: white;
  animation-name: backdiv;
  animation-duration: 1.8s; 
  animation-iteration-count: infinite;
}
@keyframes backdiv {
  50% {
    // background: #f0df81;
    box-shadow : 10px 10px 50px 15px rgb(228, 228, 172);
  }
}
@keyframes beat {
  0% {
    transform: scale(1) rotate(-45deg);
  }
  50% {
    transform: scale(0.6) rotate(-45deg);
  }
}
</style>