<template>
  <div>
    <!-- <div v-for="note in noteList" :key="note.id" class="MainList_Item">일기 하나임</div> -->
    <div align="center" class="note-empty" v-if="viewList.length === 0">
      <div>
        <div>
          <img src="../../assets/image/paper.png" style="width:90px; margin-bottom:24px;" />
        </div>
        <span>아직 작성한 일기가 없는 것 같아요.</span> <br>
        <span class="go-write-btn" @click="goWrite()">
          일기 작성하러 가기 &nbsp;&nbsp;>
        </span>
      </div>
    </div>
    <div
      v-for="note in viewList"
      :key="note.id"
      @click="goDetailNote(note)"
      class="note_Item"
    >    
      <div v-if="note.noteMediaList[0]"
        class="note_ImgBox"
        align="center"
        :style="{ 'font-family': getAllFonts[note.fontId - 1].fontUrl }"
      >
        <v-img class="note_Img" :src="note.noteMediaList[0]" /><br>
        <!-- {{note.noteContent}} -->
        <span class="note-detail-area" v-html="note.noteContent" :style="{ 'font-family': getAllFonts[note.fontId - 1].fontUrl }"></span>
      </div>
      <div v-else align="center">
        <div class="note-detail">
          <span
            class="note-detail-area"
            :style="{ 'font-family': getAllFonts[note.fontId - 1].fontUrl }"
            v-html="note.noteContent"
          >
          </span>
        </div>
      </div>
      <div class="note-under">
        <span class="note_Name">{{ note.noteTitle }}</span>
        <span class="note_Day">{{ note.noteCreatedDate }}</span>        
      </div>
    </div>
    
    <!-- 일기 페이지네이션 -->
    <div id="diary-pagination">
      <v-pagination
        style="margin-bottom:30px;"
        v-model="page"
        :length="Math.ceil(noteList.length/6)"
        @input="change"
        circle
        color="#FFB300"
      ></v-pagination>
    </div>

  </div>
</template>

<script>
import { mapGetters } from 'vuex';
export default {
  data() {
    return{
      noteList: [],
      viewList: [],
      page: 1
    }
  },
  computed: {
    ...mapGetters(['getAllFonts']),
  },
  filters: {
      test (value) {
          return value.replace("\n", "<br />")
      }
  },
  methods: {
    change(num) {
      var temp = 0
      for(let i=1;i<this.noteList.length;i++){
        if(i==num){
          this.viewList = []
          for(let i=temp;i<temp+6;i++){
            if(this.noteList.length==i)
              break
            this.noteList[i].noteContent = this.noteList[i].noteContent.replace(/\n/g, "<br />")
            this.viewList.push(this.noteList[i])
          }
        }
        temp+=6
      }
    },
    goDetailNote(note) {
      this.$store.dispatch("getDiaryContent", note.diaryId).then((res) => {
        this.$store.commit('setCurrentDiary', res.data)
        this.$store.commit('setNoteContent', note) // mutaion 호출 ('뮤테이션 이름, 매개변수)
        // this.$router.push({name: 'diary', query: {diaryId: '1', noteId: '2'}})
        this.$router.push("/diary").catch(() => {});
      })
    },
    goWrite() {
      this.$store.commit('initNoteContent')
      this.$store.commit('setIsUpdate', false)
      this.$router.push("/write").catch(() => {});
    },
  },
  created() {
    const diaryIdQuery = this.$route.query.diaryId
    const mainListModeQuery = this.$route.query.mainListMode
    if (diaryIdQuery && mainListModeQuery) {
      this.$emit('showDiaryList')
      this.$emit('selectDairy')
      return
    }
    this.$store.dispatch("noteGet").then((res) => {
      this.noteList = res.data.reverse();
      console.log(this.noteList)
      for(let i=0;i<6;i++){
        if(this.noteList.length==i) 
          break
        this.noteList[i].noteContent = this.noteList[i].noteContent.replace(/\n/g, "<br />")
        this.viewList.push(this.noteList[i])
      }
    });
    // this.$store.dispatch("getDiaryContent", this.currentDiary.id)
    //   .then((res) => {
    //     console.log(res.data)        
    //     this.noteList = res.data.note.reverse();      
    //   })
  },
};
</script>

<style lang="scss" scoped>
.note_Item {
  display: inline-block;
  width: 286px;
  height: 432px;
  margin-bottom: 31px;
  margin-left: 31px;
  // background: #aaa;
}
.note_ImgBox {
  width: 286px;
  height: 394px;
  padding: 20px;
  background: #fff;
  box-shadow: 3px 3px 11px rgba(166, 166, 168, 0.35);
  cursor: pointer;
  overflow:hidden;
  margin:0 auto;
}
.note_ImgBox:hover {
  opacity: .7;
  transition: .3s;
}
.note-detail {
  width: 286px;
  height: 394px;
  padding: 20px;
  // background: #415678;
  box-shadow: 3px 3px 11px rgba(166, 166, 168, 0.35);
  cursor: pointer;
  overflow:hidden;
  /* margin:0 auto; */
  display: flex;
  justify-content: center;
  align-items: center;
}
.note-detail:hover {
  opacity: .7;
  transition: .3s;
}
.note-detail-area {
  color: #929292;
}
.note_Img{
  // width:100%;
  object-fit: cover;
  // background-color: #eee;
}
.note_Name {
  font-size: 15px;
  max-width:190px;
  // font-weight: 500;
  cursor: pointer;
}
.note_Name:hover {
  color: #979797;
}
.note_Day{
  font-size: 15px;
  color:#929292;
}
.note-under{
  margin-top:12px;
  display: flex;
  justify-content: space-between;
  align-items:center;
}
.note-empty {
  width: 100%;
  height: 100%;
  margin: 20% 0;
  font-size: 20px;
}
.go-write-btn {
  display: inline-block;
  position: relative;
  overflow: hidden;
  cursor: pointer;
  width: 18%;
  color: #ffb319;
  margin-top: 24px;
  padding-bottom: 2px;
  font-weight: 600;
  // background-color: #eee;
}
.go-write-btn:after {
  position: absolute;
  content: '';
  width: 0;
  left: 0;
  height: 2px;
  bottom: 0px;
  background-color: #ffb319;
  // border-bottom: 1px solid #ffb319;
  transition: .25s;
}
.go-write-btn:hover {
  transition: .25s;
}
.go-write-btn:hover:after {
  width: 99%;
  left: 0;
}
</style>