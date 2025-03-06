<template>
    <div class="player-container">
      <video ref="video" controls autoplay muted playsinline></video>
      <h3>测试地址:</h3>
      <p>https://live.nodemedia.cn:8443/live/b480_264.flv</p>
      <p>https://media.w3.org/2010/05/sintel/trailer.mp4</p>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, onMounted } from "vue";
  import Hls from "hls.js";
  import flvjs from "flv.js";
  
  const video = ref<HTMLVideoElement | null>(null);
  
  function getQueryParam(name: string): string | null {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(name);
  }
  
  const videoSrc = getQueryParam("url");
  
  onMounted(() => {
    const videoEl = video.value;
    if (!videoEl || !videoSrc) return;
  
    if (videoSrc.endsWith(".m3u8")) {
      if (Hls.isSupported()) {
        const hls = new Hls();
        hls.loadSource(videoSrc);
        hls.attachMedia(videoEl);
      } else if (videoEl.canPlayType("application/vnd.apple.mpegurl")) {
        videoEl.src = videoSrc;
      }
    } else if (videoSrc.endsWith(".flv")) {
      if (flvjs.isSupported()) {
        const flvPlayer = flvjs.createPlayer({
          type: "flv",
          url: videoSrc,
          isLive: true,
        });
        flvPlayer.attachMediaElement(videoEl);
        flvPlayer.load();
        flvPlayer.play()?.catch(() => {
          console.warn("FLV 直播播放失败。");
        });
      }
    } else {
      videoEl.src = videoSrc;
    }
  
    // 只允许静音自动播放，用户手动打开声音
    videoEl.muted = true;
    videoEl.play().catch(() => {
      console.warn("自动播放失败，保持静音。");
    });
  });
  </script>
  
  <style scoped>
  body {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #000;
    margin: 0;
  }
  .player-container {
    position: relative;
    max-width: 800px;
    width: 100%;
  }
  video {
    width: 100%;
    border-radius: 10px;
  }
  </style>
  