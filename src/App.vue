<template>
  <div class="app-container">
    <InputSearch
        v-model:value="videoUrl"
        placeholder="input search text"
        size="large"
        @search="onSearch"
    >
      <template #enterButton>
        <Button @click="btnPlay()">播放</Button>
      </template>
    </InputSearch>

    <div id="rtcPlayer">
      <video id='webRtcPlayerBox' autoplay controls style="text-align:left;">
        Your browser is too old which doesn't support HTML5 video.
      </video>
    </div>
  </div>
</template>

<script setup>
import {nextTick, onUnmounted, ref} from 'vue'
import {Button, InputSearch, message} from 'ant-design-vue'

const videoUrl = ref('')
let webrtcPlayer = null

function btnPlay() {
  nextTick(() => {
    if (typeof (videoUrl.value) == "undefined" || videoUrl.value === '') {
      return message.error('请填写视频地址')
    }
    play(videoUrl.value)
  })
}

let timer = null

function play(url) {
  webrtcPlayer = new ZLMRTCClient.Endpoint({
    element: document.getElementById('webRtcPlayerBox'),// video 标签
    debug: true,// 是否打印日志
    zlmsdpUrl: url,//流地址
    simulecast: false,
    useCamera: false,
    audioEnable: true,
    videoEnable: true,
    recvOnly: true,
    usedatachannel: false,
  })
  webrtcPlayer.on(ZLMRTCClient.Events.WEBRTC_ICE_CANDIDATE_ERROR, (e) => {// ICE 协商出错
    console.error('ICE 协商出错')
    eventcallbacK("ICE ERROR", "ICE 协商出错")
  });

  webrtcPlayer.on(ZLMRTCClient.Events.WEBRTC_ON_REMOTE_STREAMS, (e) => {//获取到了远端流，可以播放
    console.log('播放成功', e.streams)
    eventcallbacK("playing", "播放成功")
  });

  webrtcPlayer.on(ZLMRTCClient.Events.WEBRTC_OFFER_ANWSER_EXCHANGE_FAILED, (e) => {// offer anwser 交换失败
    console.error('offer anwser 交换失败', e)
    eventcallbacK("OFFER ANSWER ERROR ", "offer anwser 交换失败")
    if (e.code == -400 && e.msg == "流不存在") {
      console.log("流不存在")
      timer = setTimeout(() => {
        webrtcPlayer.close();
        play(url)
      }, 100)

    }
  });

  webrtcPlayer.on(ZLMRTCClient.Events.WEBRTC_ON_LOCAL_STREAM, (s) => {// 获取到了本地流

    // document.getElementById('selfVideo').srcObject=s;
    eventcallbacK("LOCAL STREAM", "获取到了本地流")
  });

}

/**
 * 停止播放
 */
function pause() {
  if (webrtcPlayer != null) {
    webrtcPlayer.close();
    webrtcPlayer = null;
  }

}

function eventcallbacK(type, message) {
  console.log("player 事件回调", type, message)
}

onUnmounted(() => {
  clearTimeout(timer)
})

</script>

<style>
.app-container {
  min-width: 50vw;
}

#rtcPlayer {
  width: 100%;
  background: #c1c1c1;
}

#webRtcPlayerBox {
  width: 100%;
  max-height: 56vh;
  background-color: #000;
}
</style>

