<template>
  <div style="display: relative;">
    <div id="Line"></div>
    <div v-if="noteList.length > 0">    
      <div      
        id="diary-list-item"
        v-for="note in viewList"
        :key="note.noteId"
      >
        <div id="diary-wrap">
          <div class="diary-content">
            <div class="diary-title-wrap">
              <div class="title-sec">
                <span class="diary-title" :style="{ 'font-family': getAllFonts[note.fontId - 1].fontUrl }">{{ note.noteTitle }}</span>
              </div>
              <div class="date-sec">
                <span class="diary-writer">{{ note.writerNickName}}</span>
                <span class="diary-date">
                  {{ note.noteCreatedDate.slice(2, 4)}}.{{ note.noteCreatedDate.slice(5, 7)}}.{{ note.noteCreatedDate.slice(8, 10)}}
                </span>
              </div>
            </div>
            <div id="horizon-line"></div>
            <div class="diary-text">
              <span :style="{ 'font-family': getAllFonts[note.fontId - 1].fontUrl }" v-html="note.noteContent"></span>
            </div>
            <div class="diary-hashtag" v-for="(hashtag, idx) in note.noteHashtagList" :key="idx">
              <span>#{{ hashtag }}</span>
            </div>
            <div class="diary-img-wrap">
              <div v-if="note.noteMediaList.length > 1">
                <v-carousel
                  height="300"
                  hide-delimiters
                  :show-arrows="true"
                  style="height:300px;">
                  <v-carousel-item
                    v-for="(img, i) in note.noteMediaList"
                    :key="i"
                    :src="img"
                    class="diary-content-img"
                  ></v-carousel-item>
                  <img v-for="sticker in note.stickerList" :key="sticker.id" :src="sticker.sticker.stickerUrl" :style="{top:sticker.topPixel, left:sticker.leftPixel}" class="sticker"/>
                </v-carousel>
              </div>
              <div v-else-if="note.noteMediaList.length == 0">
              </div>
              <div v-else>
                <div style="position:relative;">
                  <img :src="note.noteMediaList[0]" class="diary-content-img" alt="?????? ??????" style="width:371px;"/>
                  <img v-for="sticker in note.stickerList" :key="sticker.id" :src="sticker.sticker.stickerUrl" :style="{top:sticker.topPixel, left:sticker.leftPixel}" class="sticker"/>
                </div> 
              </div>
            </div>
          </div>
          <div class="diary-emotion">
            <div class="emotion-left">
              <div class="emo-item">
                <v-icon @click="clickHeart(note.noteId, note.emotionStatusRes.pressLike)" class="emo-icon" :class="{ 'like-clicked' : note.emotionStatusRes.pressLike }">favorite_border</v-icon>
                <div class="emotion-cnt">
                  <span>{{note.emotionStatusRes.likeEmotionCount}}</span>
                </div>
              </div>
              <div class="emo-item">
                <v-icon @click="clickSmile(note.noteId, note.emotionStatusRes.pressFunny)" class="emo-icon" :class="{ 'funny-clicked' : note.emotionStatusRes.pressFunny }">sentiment_very_satisfied</v-icon>
                <div class="emotion-cnt">
                  <span>{{note.emotionStatusRes.funnyEmotionCount}}</span>
                </div>
              </div>
              <div class="emo-item">
                <v-icon @click="clickSad(note.noteId, note.emotionStatusRes.pressSad)" class="emo-icon" :class="{ 'sad-clicked' : note.emotionStatusRes.pressSad }">sentiment_dissatisfied</v-icon>
                <div class="emotion-cnt">
                  <span>{{note.emotionStatusRes.sadEmotionCount}}</span>
                </div>
              </div>
            </div>
            <div class="emotion-right" v-if="note.writerId == loginUser.userId">
              <span @click="goUpdate(note)">??????</span>
              <span @click="onDialog(note)">??????</span>
            </div>
            <div class="emotion-right" v-else>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else id="diary-empty">
      <div>
        <img src="../../assets/image/paper.png" style="width:90px; margin-bottom:24px;" />
      </div>
      <span>?????? ????????? ????????? ?????? ??? ?????????.</span> <br>
      <span class="go-write-btn" @click="goWrite()">
        ?????? ???????????? ?????? &nbsp;&nbsp;>
      </span>
    </div>
    <!-- ?????? ?????????????????? -->
    <div id="diary-pagination">
      <v-pagination
        v-model="page"
        :length="Math.ceil(noteList.length/2)"
        @input="change"
        circle
        color="#FFB300"
        class="page-sec"
      ></v-pagination>
    </div>
    <!-- Dialog -->
    <v-dialog v-model="dialog" persistent max-width="286">
      <v-card id="Dialog">
        <div id="Dialog_Header">
          <v-icon @click="dialog = false" id="Dialog_Close" style="font-size: 1.5em">close</v-icon>
        </div>
        <div id="Dialog_Content">
          <div id="Dialog_Text">
            ????????? <span style="color: #ffb319">??????</span>???????????????????
          </div>
          <div id="Dialog_Btn_Box">
            <v-btn @click="onDeleteNote(note.noteId)" style="background: #ffb319; color: white" class="Dialog_Btn">??????
            </v-btn>
            <v-btn @click="dialog = false" style="background: #9f9f9f; color: white" class="Dialog_Btn">??????</v-btn>
          </div>
        </div>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
  import {mapState, mapGetters} from 'vuex';

  export default {
    data() {
      return {
        page: 1,
        dialog: false,
        noteList: [],
        viewList: [],
        hashtagList: [],
        note: [],
        emotionReq: {
          "emotionInfoId": 0,
          "noteId": 0,
          "diaryId": 0,
        },
      };
    },
    methods: {
      change(num){
      var temp = 0
        for(let i=1;i<=this.noteList.length;i++){
          if(i==num){
            this.viewList = []
            if(this.noteList.length>=temp+1){
              this.viewList.push(this.noteList[temp])
            }
            if(this.noteList.length>=temp+2){
              this.viewList.push(this.noteList[temp+1]) 
            }
            break
          }
          temp+=2
        }
      },
      clickHeart(noteId, isPress) {
        this.emotionReq.emotionInfoId = 1
        this.emotionReq.noteId = noteId
        if (!isPress) {
          this.emotionConfirm()
        } else {
          this.emotionCancel()
        }
      },
      clickSmile(noteId, isPress) {
        this.emotionReq.emotionInfoId = 2
        this.emotionReq.noteId = noteId
        if (!isPress) {
          this.emotionConfirm()
        } else {
          this.emotionCancel()
        }
      },
      clickSad(noteId, isPress) {
        this.emotionReq.emotionInfoId = 3
        this.emotionReq.noteId = noteId
        if (!isPress) {
          this.emotionConfirm()
        } else {
          this.emotionCancel()
        }
      },
      emotionConfirm() {
        this.$store.dispatch("emotionConfirm", this.emotionReq)
          .then(() => {
            this.$store.dispatch("getDiaryContent", this.currentDiary.id)
              .then((res) => {
                this.noteList = res.data.note.reverse();
                for (let i = 0; i < this.noteList.length; i++) {
                  this.noteList[i].noteContent = this.noteList[i].noteContent.replace(/\n/g, "<br />")
                }
                this.emotionReq.diaryId = this.currentDiary.id
                this.change(this.page)
              })
          })
      },
      emotionCancel() {
        this.$store.dispatch("emotionCancel", this.emotionReq)
          .then(() => {
            this.$store.dispatch("getDiaryContent", this.currentDiary.id)
              .then((res) => {
                this.noteList = res.data.note.reverse();
                for (let i = 0; i < this.noteList.length; i++) {
                  this.noteList[i].noteContent = this.noteList[i].noteContent.replace(/\n/g, "<br />")
                }
                this.emotionReq.diaryId = this.currentDiary.id
                this.change(this.page)
              })
          })
      },
      goWrite() {
        this.$store.commit('initNoteContent')
        this.$store.commit('setIsUpdate', false)
        this.$router.push("/write").catch(() => {});
      },
      goUpdate(note) {
        const localNote = {
          noteId: note.noteId,
          diaryId: note.diaryId,
          fontId: note.fontId,
          layoutId: 1,
          designId: 1,
          writerId: note.writerId,
          noteTitle: note.noteTitle,
          noteContent: note.noteContent,
          noteS3MediaList: note.noteMediaList,
          noteMediaList: [],
          noteHashtagList: '#' + note.noteHashtagList[0],
          stickerList: note.stickerList,
          emotionList: note.emotionList
        }
        
        localNote.noteContent = note.noteContent.replace(/\/>/g, "")
        localNote.noteContent = localNote.noteContent.replace(/<br /g, "\n") // ??? ?????? ?????? html to string
        const stickerList = [];
        for(let i = 0; i < localNote.stickerList.length; i++){
          stickerList.push({
            stickerId : localNote.stickerList[i].sticker.id,
            topPixel : localNote.stickerList[i].topPixel,
            leftPixel : localNote.stickerList[i].leftPixel
          })
        }
        this.$store.commit('setStickerList', note.stickerList)
        localNote.stickerList = stickerList;
        for (let i = 1; i < note.noteHashtagList.length; i++) {
          localNote.noteHashtagList += ('#' + note.noteHashtagList[i])
        }
        this.$store.commit('setNoteContent', localNote)
        this.$store.commit('setIsUpdate', true)
        this.$router.push("/write");
      },
      onDialog(note) {
        this.note = note;
        this.dialog = true;
      },
      onDeleteNote(id) {
        this.$store.dispatch("deleteNote", id)
          .then(() => {
            this.dialog = false;
            this.$store.dispatch("getDiaryContent", this.currentDiary.id)
              .then((res) => {
                this.noteList = res.data.note.reverse();
                this.emotionReq.diaryId = this.noteList[0].diaryId
                this.page = 1
                this.viewList = []
                if(this.noteList.length>=1){
                  this.viewList.push(this.noteList[0])
                }
                if(this.noteList.length>=2){
                  this.viewList.push(this.noteList[1])
                }
              })
          })
      },
    },
    computed: {
      currentDiary() {
        return this.$store.getters.getCurrentDiary;
      },
      loginUser() {
        return this.$store.getters.getLoginUser;
      },
      ...mapState(['loginUser']),
      ...mapGetters(['getAllFonts']),
      noteContent() {
        return this.$store.getters.getNoteContent;
      },
      ...mapState([
        'loginUser'
      ]),
    },
    created() {
      // this.$store.dispatch("getDiaryContent", this.currentDiary.id) // ????????? notelist??? ???????????? ????????? ???????????? 
      //   .then((res) => {
      //     this.noteList = res.data.note.reverse();
      //     // ????????? ?????? id?????? ?????? ??????url????????? ??????
      //     for (var j=0; j<this.noteList.length; j++) {
      //       for (var i = 0; i < this.getAllFonts.length; i++) {
      //         if (this.getAllFonts[i].id == this.noteList[j].fontId) {
      //           this.noteList[j].fontId = this.getAllFonts[i].fontUrl
      //           break
      //         }
      //       }
      //     }
      //   })
      const diaryIdQuery = this.$route.query.diaryId
      const noteIdQuery = this.$route.query.noteId
      if (diaryIdQuery && noteIdQuery) {
        // alert('???????????? ???????????? ' + diaryIdQuery)
        // alert('?????? ???????????? ' + noteIdQuery)
        this.$store.dispatch("getDiaryContent", diaryIdQuery)
        .then((res) => {
          this.noteList = res.data.note.reverse();
          for (let i = 0; i < this.noteList.length; i++) {
            this.noteList[i].noteContent = this.noteList[i].noteContent.replace(/\n/g, "<br />")
          }
          this.emotionReq.diaryId = this.noteList[0].diaryId

          for (let i = 0; i < this.noteList.length; i++) {
            if (this.noteList[i].noteId === noteIdQuery) {
              this.page = parseInt(i / 2) + 1
              this.change(this.page)
              break
            }
          }
        })
        return
      }
      
      this.$store.dispatch("getDiaryContent", this.currentDiary.id)
        .then((res) => {
          this.noteList = res.data.note.reverse();
          this.emotionReq.diaryId = this.currentDiary.id
          
          for (let i = 0; i < this.noteList.length; i++) {
            this.noteList[i].noteContent = this.noteList[i].noteContent.replace(/\n/g, "<br />")
          }
          this.viewList = []
          var page = 1
          for(let i=0;i<this.noteList.length;i++){
            if(i>0 && i%2==0){
              page++
            }
            
            if(this.noteContent.noteId == this.noteList[i].noteId) {
              if(i==this.noteList.length-1 && i%2==0){
                this.viewList.push(this.noteList[i])
              }
              else if(i%2==0 && i<this.noteList.length-1){
                this.viewList.push(this.noteList[i])
                this.viewList.push(this.noteList[i+1])
              }
              else{
                this.viewList.push(this.noteList[i-1])
                this.viewList.push(this.noteList[i])
              }
              this.page = page

              break
            }
          }
          
          // if(this.noteList.length>=1){
          //   this.viewList.push(this.noteList[0])
          // }
          // if(this.noteList.length>=2){
          //   this.viewList.push(this.noteList[1])
          // }
        })
    },
  };
</script>

<style lang="scss" scoped>
#Line {
  height: 590px;
  border: 1px solid #e7e7e7;
  position: absolute;
  top: 71px;
  left: 461px;
}
#horizon-line {
  width: 100%;
  border-bottom: 1px solid #ccc;
  margin-bottom: 8px;
}
#diary-empty {
  width: 50%;
  font-size: 18px;
  text-align: center;
  margin-top: 140px;

  // background-color: #ffb319;
  span {
    color: #444;
  }

  .go-write-btn {
    display: inline-block;
    position: relative;
    overflow: hidden;
    cursor: pointer;
    width: 37%;
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
    width: 98%;
    left: 0;
  }
}
#diary-list-item {
  float: left;
  // background: #ccc;
}

#diary-list-item::after {
  // content: '';
  clear: both;
  display: block;
}

#diary-wrap {
  width: 371px;
  height: 538px;
  margin: 36px 36px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  // background: #ffb319;
}
.diary-content {
  // min-height: 480px;
  max-height: 485px;
  // background-color: green;
  overflow: hidden;
}
.diary-title-wrap {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 3px;
}
.diary-title {
  font-size: 20px;
  font-weight: 500;
  line-height: 1.2;
}
.date-sec {
  // background-color: aquamarine;
  // margin-left: 3px;
  min-width: 90px;
  text-align: right;
}
.diary-writer {
  font-size: 14px;
  margin-right: 12px;
  // background-color: green;
}
.diary-date {
  color: #929292;
  font-size: 14px;
  // background-color: lightblue;
}
.diary-text {
  margin-bottom: 12px;
  // background-color: lightblue;
  overflow-x: hidden;
  overflow-y: auto;
  white-space:nowrap;
  max-height: 95px;
  line-height: 1.4;
}
.diary-text::-webkit-scrollbar-button {
  display: none;
}
.diary-text::-webkit-scrollbar {
  height: 0px;
  width: 5px;
  cursor: pointer;
}
.diary-text::-webkit-scrollbar-thumb {
  background-color: #b9b9b9;
  border-radius: 10px;
}
.diary-text::-webkit-scrollbar-track {
  background-color: #e2e2e2;
  border-radius: 10px;
}
.diary-hashtag {
  display: inline-block;
  margin-right: 5px;
  font-weight: 500;

  span {
    color: #ffb319;
  }
}
.diary-img-wrap {
  margin-top: 17px;
  min-height: 250px;
  // max-height: 300px;
  overflow: hidden;
  // background-color: #f7f7f7;
}
.diary-content-img {
  height: 300px;
  object-fit: cover;
  // background-color: aquamarine;
}
.diary-emotion {
  display: flex;
  justify-content: space-between;
}
.emotion-left {
  color: #929292;
  // background-color:cornflowerblue;
}
.emo-item {
  display: inline-block;
}
.emo-icon {
  cursor: pointer;
}
.emotion-cnt {
  display: inline-block;
  margin: 0 10px 0 5px;

  span {
    color: #929292;
  }
}
.emotion-right {
  display: inline-block;

  span {
    font-size: 14px;
    color: #9f9f9f;
    margin-left: 13px;
    cursor: pointer;
  }
  span:hover {
    color: #222;
  }
}
#diary-pagination {  
  position: absolute;
  // margin-top: 50px;
  left: auto;
  bottom: -65px;
  width: 886px;
  overflow: hidden;
  // background-color: #eee;
  // margin-bottom: 20px;
}
.page-sec {  
  width: 100%;
  height: 50px;
  }
#Dialog {
  height: 180px;
}
#Dialog_Header {
  height: 40px;
}
#Dialog_Close {
  float: right;
  margin-top: 5px;
  margin-right: 5px;
}
#Dialog_Text {
  width: 100%;
  height: 38px;
  line-height: 38px;
  // background-color: #eee;
  margin: 0 auto;
  margin-top: 5px;
  font-size: 16px;
  text-align: center;
}
#Dialog_Btn_Box {
  width: 164px;
  height: 32px;
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  margin-top: 25px;
}
.Dialog_Btn {
  width: 76px;
  height: 32px;
  color: #585858;
  box-shadow: none;
}
.like-clicked {
  color: #E41D35;
}
.funny-clicked {
  color: #21B74B;
}
.sad-clicked {
  color: #385FC7;
}
.sticker{
  width:50px; 
  height:50px; 
  position:absolute;
  z-index:100;
}
</style>