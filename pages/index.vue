<template>
  <div class="mostParent">
    <div class="topbar">
      <div class="topbar-title">聖光祭 展示団体入退場管理システム</div>
      <select class="display-select" v-model="selectedDisplay">
        <option v-for="name in Object.keys(displayList)" :key="name">{{ name }}</option>
      </select>
      <div class="mode">
        {{ (isInType == "in")? "入場" : "退場" }}
      </div>
    </div>
    <div class="main">
      <div v-if="showCapture" class="capture-qr-content">
        <qrcode-stream :class="{ 'capture-qr-screen': true, 'in': isInType }" v-if="isCapture && selectedDisplay != ''" @decode="onDecode" @init="onInit" />
      </div>
      <div v-else class="capture-result">
        <div class="capture-result-innner">
          <img v-if="isOK" src="@/assets/img/OK.png"/>
          <img v-else src="@/assets/img/NG.png"/><br>
          <span v-html="resultMes"></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script >
import Vue from 'vue'
import VueQrcodeReader from 'vue-qrcode-reader'
Vue.use(VueQrcodeReader)

export default Vue.extend({
  data() {
    return {
      paused: false,
      error: '',
      result: '',
      isCapture: true,
      showCapture: true,
      iconPath: "",
      resultMes: "",
      isOK: false,
      displayList: {'生物部': 'Dx01', '地歴部': 'Dx02', 'ジョージ研': 'Dx03', 'リア盤': 'Dx04', 'ボドゲ': 'Dx05', 'コーヒーカップ': 'Dx06', 'ボルぽこ': 'Dx07', 'ぎゃんぶる': 'Dx08', '地天部': 'Dx09', 'ジョニー': 'Dx10', '技術展示': 'Dx11', 'ディズニー': 'Dx12', 'エース': 'Dx13', '魔女卓': 'Dx14', '古本市': 'Dx15', '聖光道場': 'Dx16', 'ごと研': 'Dx17', '数研': 'Dx18', '英語劇': 'Dx19', '交研': 'Dx20', '宇宙': 'Dx21', 'プロ研': 'Dx22', 'アイロジ': 'Dx23', 'トリハ': 'Dx24', '喫茶': 'Dx25', 'クイ研': 'Dx26', '物科部': 'Dx27', '文芸': 'Dx28', 'ポケセン': 'Dx29', '美術部': 'Dx30', 'コン部': 'Dx31', '駅弁': 'Dx32', 'かるた': 'Dx33', 'ぶいえいす': 'Dx34'},
      selectedDisplay: '',
      isInType: false
    }
  },
  mounted() {
    this.isInType = (this.$route.query.type == "in")
  },
  methods: {
    onDecode (result) {
      this.result = result
      this.isCapture = false
      this.callAppsScript("editCongestionDegree",
       {userId: this.result, displayId: this.displayList[this.selectedDisplay], body: (this.$route.query.type == "in")? "plus" : "minus"},
       res => {
        console.log("return")
        this.isOK = false
        if (res.status == "ok") {
          console.log("test")
          this.resultMes = "認証に成功しました。お進みください。"
          this.isOK = true
        } else if (res.status == "max") {
          this.resultMes = "室内は満員です。入場はお控えください。"
        } else if (res.status == "timeout") {
          this.resultMes = "4時間程度のご滞在にご協力お願いします。"
        }
        this.showCapture = false
        setTimeout(() => {
          this.showCapture = true
          this.isCapture = true
        }, 4000)
      })
    },
    callAppsScript(functionName, parameters, callbackFunc) {
      var request = { 'function': functionName, 'parameters': parameters };
      $.ajax({
        type: 'POST',
        url: "https://script.google.com/macros/s/AKfycbxvUDxQ4VNhzp07091szeG1L8unsnFHxA34jFmu0PCCoO0P4Dw2l8JH3aMSKwMEu6gaZg/exec",
        data: request,
        cache: false,
        dataType: "jsonp",
        success: function(data) {
          callbackFunc(data);
        }
      });
    },
    async onInit (promise) {
      try {
        await promise
      } catch (error) {
        if (error.name === 'NotAllowedError') {
          this.error = "ERROR: you need to grant camera access permisson"
        } else if (error.name === 'NotFoundError') {
          this.error = "ERROR: no camera on this device"
        } else if (error.name === 'NotSupportedError') {
          this.error = "ERROR: secure context required (HTTPS, localhost)"
        } else if (error.name === 'NotReadableError') {
          this.error = "ERROR: is the camera already in use?"
        } else if (error.name === 'OverconstrainedError') {
          this.error = "ERROR: installed cameras are not suitable"
        } else if (error.name === 'StreamApiNotSupportedError') {
          this.error = "ERROR: Stream API is not supported in this browser"
        }
      }
    }
  }
})
</script>


