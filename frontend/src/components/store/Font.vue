<template>
  <div>
    <div class="input-area">
      <v-text-field
        v-model="previewText"
        color="#FFB319"
        placeholder="이곳에 텍스트를 입력해보세요"
      >
      </v-text-field>
    </div>
    <div v-for="font in viewList" :key="font.id" class="Font_Item">
      <div class="font-header">
        <div class="font-name">
          <span :style="{ 'font-family': font.fontUrl }">{{
            font.fontName
          }}</span>
        </div>
        <div class="font-price">
          <span >{{ font.fontPrice }}장</span>
        </div>
      </div>
      <div class="Font_Content">
        <div class="Font_Discription" :style="{ 'font-family': font.fontUrl }">
          <!-- 즐거운 일기 쓰기! papers에서 매일매일 추억을 쌓아보아요. -->
          {{previewText}}
        </div>
        <div v-if="font.owned">
          <v-btn
            @click="[(dialog = true), sendInfo(font.fontPrice, font.id)]"
            class="Font_Btn"
            color="#FFB319"
            outlined
            disabled
            >보유중</v-btn
          >
        </div>
        <div v-else>
          <v-btn
            @click="[(dialog = true), sendInfo(font.fontPrice, font.id)]"
            class="Font_Btn"
            color="#FFB319"
            outlined
            >구매</v-btn
          >
        </div>
      </div>
    </div>
    <!-- 폰트 페이지네이션 -->
    <div id="diary-pagination">
      <v-pagination
        style="margin-bottom: 30px;"
        v-model="page"
        :length="Math.ceil(fontList.length/9)"
        @input="change"
        circle
        color="#FFB300"
      ></v-pagination>
    </div>
    <!-- Dialog -->
    <v-dialog v-model="dialog" persistent max-width="286">
      <v-card id="Dialog">
        <div id="Dialog_Header">
          <v-icon
            @click="dialog = false"
            id="Dialog_Close"
            style="font-size: 1.5em"
            >close</v-icon
          >
        </div>
        <div id="Dialog_Content">
          <div id="Dialog_Text">
            종이 <span style="color: #ffb319">{{buyFontPrice}}장</span>을
            소비해 구매하시겠습니까?
          </div>
          <div id="Dialog_Btn_Box">
            <v-btn style="background: #ffb319; color: white" class="Dialog_Btn" @click="buyFont(buyFontId)"
              >구매</v-btn
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
      page: 1,
      dialog: false,
      fontList: [],
      viewList: [],
      buyFontPrice: null,
      buyFontId: null,
      loginUser: {
        userMileage: '',
      },
      myFontList: [],
      previewText: '즐거운 일기 쓰기! papers에서 매일매일 추억을 쌓아보아요.',
    };
  },
  created () {
    this.getAllFonts()
    this.loginUser = this.$store.getters['getLoginUser'];
  },
  methods: {
    change(num) {
      var temp = 0
      for(let i=1;i<this.fontList.length;i++){
        if(i==num){
          this.viewList = []
          for(let i=temp;i<temp+9;i++){
            if(this.fontList.length==i)
              break
            this.viewList.push(this.fontList[i])
          }
        }
        temp+=9
      }
    },
    getAllFonts: function () {
      this.$store.dispatch("getAllFonts")
      .then((res) => {
        this.fontList = res.data
        console.log(this.fontList, '폰트리스트')
  
        for(let i=0;i<9;i++){
          if(this.fontList.length==i) 
              break
            this.viewList.push(this.fontList[i])
        }
      })
    },
    getAllFonts2: function (page) {
      this.$store.dispatch("getAllFonts")
      .then((res) => {
        this.fontList = res.data
        this.change(page)
      })
    },
    sendInfo: function (price, id) {
      console.log(price, '클릭한 폰트 가격')
      console.log(id, '클릭한 폰트 아이디')
      this.buyFontPrice = price
      this.buyFontId = id
    },
    buyFont(fontId) {
      console.log(fontId, '구매하려는 폰트 아이디')
      this.$store.dispatch("buyFont", fontId)
      .then((res) => {
        console.log(res)
        this.loginUser.userMileage -= this.buyFontPrice
        this.$store.commit('setLoginUser', this.loginUser)
        this.viewList = []
        this.getAllFonts2(this.page)
        // this.$router.go()
        Swal.fire({
          icon: "success",
          title:
            '<span style="font-size:25px;">성공적으로 구매되었습니다.</span>',
          confirmButtonColor: "#b0da9b",
          confirmButtonText: '<span style="font-size:18px;">확인</span>',
        });
        this.dialog = false
      })
      .catch(() => {
        Swal.fire({
          icon: "error",
          title:
            '<span style="font-size:25px;">마일리지가 부족합니다.</span>',
          confirmButtonColor: "#f27474",
          confirmButtonText: '<span style="font-size:18px;">확인</span>',
        });
      })
    }
  },
};
</script>

<style lang="scss" scoped>
.Font_Item {
  display: inline-block;
  width: 286px;
  height: 180px;
  margin-bottom: 32px;
  margin-left: 32px;
  border-radius: 8px;
  box-shadow: 3px 3px 11px rgba(166, 166, 168, 0.25);
  overflow: hidden;
}
.font-header {
  height: 50px;
  border-bottom: 1px solid #e7e7e7;
  // background-color: antiquewhite;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
/* .Font_Content {
  height: 129px;
} */
.font-name {
  margin-left: 16px;
  font-size: 16px;
  font-weight: 500;
  // background-color: lightcoral;
  span {
    // background: yellow;
    line-height: 1;
  }
}
.font-price {
  font-size: 16px;
  width: 40px;
  margin-right: 16px;  
  font-weight: 600;
  // background: #ccc;
  text-align: right;
  span {
    color: #ffb319;
  }
}
.Font_Discription {
  /* width: 243px; */
  height: 62px;
  margin: 16px 16px 0 16px;
  font-size: 15px;
  color: #585858;
  // background-color: aquamarine;
  overflow: hidden;
}
.Font_Btn {
  width: 70px;
  height: 32px;
  margin-top: 4px;
  margin-right: 16px;
  float: right;
  font-size: 15px;
  box-shadow: none;
  /* color: #585858; */
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
  width: 130px;
  // height: 38px;
  margin: 0 auto;
  margin-top: 5px;
  font-size: 16px;
  line-height: 1.4;
  text-align: center;
  // background: #585858;
}
#Dialog_Btn_Box {
  width: 164px;
  height: 32px;
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  margin-top: 24px;
}
.Dialog_Btn {
  width: 76px;
  height: 32px;
  color: #585858;
  box-shadow: none;
}
.input-area {
  width: 45%; 
  margin-left: 34px; 
  margin-top: -20px;
}
</style>