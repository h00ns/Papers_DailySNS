<template>
  <div id="WriteContent_Container">
    <div id="WriteContent_Templete">
      <v-combobox
        v-model="selectDiary"
        :items="diaryTitleList"
        label="일기장 선택"
      ></v-combobox>
      <v-text-field
        label="제목입력란"
        color="#FFB319"
        v-model="noteContent.noteTitle"
      ></v-text-field>
      <v-textarea
          solo
          name="input-7-4"
          rows="15"
          label="내용입력란"
          v-model="noteContent.noteContent"
        ></v-textarea>
      <v-file-input
        v-model="noteContent.noteMedia"
        multiple
        small-chips
        truncate-length="15"
      ></v-file-input>
    </div>
    <div id="HashTag_Input">
      <v-text-field
        v-model="noteContent.noteHashtag"
        label="#여기에 #해시태그를 #입력하세요"
        color="#FFB319"
      ></v-text-field>
    </div>
    <div id="WriteContent_Btn">
      <v-btn @click="updateNote" id="Write_Btn">수정</v-btn>
      <v-btn @click="back" id="Back_Btn"> 취소 </v-btn>
    </div>
  </div>
</template>

<script>
import Swal from "sweetalert2";
import { mapState } from 'vuex';

export default {
  props: {
    note: Object,
  },
  data() {
    return {
      diaryTitleList:[],
      diaryList:[],
      selectDiary:'',
    }
  },
  computed: {
    loginUser() {
      return this.$store.getters.getLoginUser;
    },
    currentDiary() {
      return this.$store.getters.getCurrentDiary;
    },
    ...mapState([
      'noteContent'
    ]),

  },
  methods: {
    back() {
      this.$router.go(-1);
    },
    updateNote() {
      var selectDiaryId = -1 
      for(let i=0;i<this.diaryList.length;i++){
        if(this.selectDiary == this.diaryList[i].diaryTitle){
          selectDiaryId = this.diaryList[i].id
          break
        }
      }
      const tmp = this.noteHashtag.split(",")
      const noteHashtagList = []
      for(let i=1;i<tmp.length;i++){
        noteHashtagList[i-1] = tmp[i]
      }

      // const note = {
      //   designId: 1,
      //   diaryId: selectDiaryId,
      //   emotionList:[],
      //   fontId: 1,
      //   layoutId: 1,
      //   noteContent: this.noteContent,
      //   noteHashtagList: noteHashtagList,
      //   noteS3MediaList: [],
      //   noteMediaList: this.noteMedia,
      //   noteTitle: this.noteTitle,
      //   stickerList: [
      //     {
      //       leftPixel: 1,
      //       stickerId: 1,
      //       topPixel: 1,
      //     },
      //   ],
      //   writerId: this.loginUser.userNickname,
      // }
      // console.log(note)

      const formData = new FormData()
      formData.append('designId', 1)
      formData.append('diaryId', selectDiaryId)
      formData.append('emotionList.writerId[]', '')
      formData.append('emotionList.emotionInfoId[]', '')
      formData.append('emotionList.noteId[]', '')
      formData.append('fontId', 1)
      formData.append('layoutId', 1)
      formData.append('noteContent', this.noteContent)
      formData.append('noteHashtagList', noteHashtagList)
      formData.append('noteS3MediaList[]', null)
      for(let i = 0; i < this.noteMedia.length; i++){
        formData.append('noteMediaList[]', this.noteMedia[i])  
      }
      formData.append('noteTitle', this.noteTitle)
      
      for(let i = 0; i < this.stickerList.length; i++){
        formData.append('stickerList.leftPixel[]', this.stickerList[i].leftPixel)
        formData.append('stickerList.stickerId[]', this.stickerList[i].stickerId)
        formData.append('stickerList.topPixel[]', this.stickerList[i].topPixel)
      }
      
      formData.append('writerId', this.loginUser.userNickname)

      console.log(formData)
      this.$store.dispatch("modifyNote", formData).then(() => {
        Swal.fire({
            icon: "success",
            title:
              '<span style="font-size:25px;">일기 수정 완료.</span>',
            confirmButtonColor: "#b0da9b",
            confirmButtonText: '<span style="font-size:18px;">확인</span>',
          });
        this.$router.push('diarylist1').catch(() => {})
      });
    },
  },
  created() {
      this.selectDiary = this.currentDiary.diaryTitle

      this.$store.dispatch("diaryGet").then((res) => {
        const tmp = []
        for(let i=0;i<res.data.length;i++) {
          tmp[i] = res.data[i].diaryTitle
        }
        this.diaryTitleList = tmp
        console.log(tmp)
        this.diaryList = res.data
      });
    } 
};
</script>

<style lang="scss" scoped>
#WriteContent_Container {
  width: 530px;
  height: 854px;
}
#WriteContent_Templete {
  height: 684px;
  box-shadow: 3px 3px 11px rgba(166, 166, 168, 0.25);
}
#HashTag_Input {
  margin-top: 30px;
}
#WriteContent_Btn {
  height: 38px;
  width: 152px;
  margin: 0 auto;
  margin-top: 30px;
  box-shadow: none;
}
#Write_Btn {
  background: #ffb319;
  color: white;
  font-size: 17px;
  width: 70px;
  box-shadow: none;
}
#Back_Btn {
  background: #9f9f9f;
  color: white;
  font-size: 17px;
  width: 70px;
  margin-left: 12px;
  box-shadow: none;
}
</style>