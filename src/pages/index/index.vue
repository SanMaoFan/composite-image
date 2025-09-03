<template>
  <view class="container">
    <view class="user-container">
      <view class="img-container">
        <canvas class="canvas" canvas-id="canvasContainer"></canvas>

        <!-- 用户选择图片 -->
        <image class="" :src="curImgData" mode="aspectFill"></image>
        <!-- 上层的图片 -->
        <image
          class="cover-img"
          :src="curCoverImgData"
          mode="aspectFill"
        ></image>
      </view>
      <view class="btn-group">
        <button
          size="mini"
          type="default"
          open-type="chooseAvatar"
          @chooseavatar="chooseImg"
        >
          选择图片
        </button>
        <button size="mini" type="default" @click="downloadImage">
          下载图片
        </button>
      </view>
    </view>
    <!-- 供选择的图片 -->
    <scroll-view class="scroll-container" scroll-y>
      <view class="list-container">
        <view class="list-item" v-for="item in imgList" :key="item.id">
          <view class="img-container">
            <image
              class="img"
              :src="item.img"
              mode="aspectFill"
              @click="chooseCoverImg(item.img, item.id)"
            />
            <icon
              v-show="item.id === curCoverImgId"
              class="check"
              type="success"
              color="green"
              size="26"
            ></icon>
          </view>
        </view>
      </view>
    </scroll-view>
  </view>
</template>

<script setup>
import { onMounted, nextTick, reactive, ref } from "vue";
import image2 from "../../static/cover/2.png";
import image3 from "../../static/cover/3.png";

const imgList = reactive([
  {
    id: 2,
    img: image2,
  },
  {
    id: 3,
    img: image3,
  },
]);

// 用户选择当前上层图片id
const curCoverImgId = ref("");
// 用户选择当前上层图片数据
const curCoverImgData = ref("");

// 用户选择本地的图片
const curImgData = ref("");

// 画布
const canvas = ref(null);
// 画布宽高
const canvasHeight = ref();
const canvasWidth = ref();

onMounted(() => {
  canvas.value = uni.createCanvasContext("canvasContainer");
  const query = uni
    .createSelectorQuery()
    .select(".canvas")
    .boundingClientRect()
    .exec((res) => {
      const [{ width, height }] = res;
      canvasWidth.value = width;
      canvasHeight.value = height;
    });

  console.log(canvas.value.height);
});

// 选择用户本地图片
function chooseImg(e) {
  const { avatarUrl } = e.detail;
  curImgData.value = avatarUrl;
  canvas.value.drawImage(
    avatarUrl,
    0,
    0,
    canvasWidth.value,
    canvasHeight.value
  );
  // 进行用户系统本地取用图片
  // uni.chooseMedia({
  //   count: 1,
  //   mediaType: ["image"],
  //   sourceType: ["album", "camera"],
  //   camera: "back",
  //   success(res) {
  //     const [{ tempFilePath }] = res.tempFiles;
  //     curImgData.value = tempFilePath;
  //     canvas.value.drawImage(
  //       tempFilePath,
  //       0,
  //       0,
  //       canvasWidth.value,
  //       canvasHeight.value
  //     );
  //   },
  //   fail(res) {
  //     console.log("失败", res);
  //   },
  //   complete(res) {
  //     console.log("完成", res);
  //   },
  // });
}

// 选择上层的图片
function chooseCoverImg(img, id) {
  curCoverImgId.value = id;
  curCoverImgData.value = img;
}

// 下载图片
function downloadImage() {
  if (!curCoverImgId.value && !curImgData.value) {
    uni.showToast({
      title: "请上传本地图片和选择上层图片",
      icon: "warning",
      duration: 2000,
    });
    return;
  }
  // 合成图片
  // 绘制用户上传或者头像
  canvas.value.drawImage(
    curImgData.value,
    0,
    0,
    canvasWidth.value,
    canvasHeight.value
  );
  // 绘制选择的上层图片
  canvas.value.drawImage(
    curCoverImgData.value,
    0,
    0,
    canvasWidth.value,
    canvasHeight.value
  );
  canvas.value.draw(false, () => {
    // 下载
    uni.canvasToTempFilePath({
      canvasId: "canvasContainer",
      success(res) {
        uni.saveImageToPhotosAlbum({
          filePath: res.tempFilePath,
          success() {
            uni.showToast({
              title: "保存成功",
              icon: "success",
              duration: 2000,
            });
          },
          fail(err) {
            uni.showToast({
              title: "保存失败",
              icon: "error",
              duration: 2000,
            });
            console.log("save fail: ", err);
          },
        });
      },
      fail(err) {
        uni.showToast({
          title: "下载失败",
          icon: "error",
          duration: 2000,
        });
        console.log("download fail: ", err);
      },
    });
  });
}
</script>

<style scoped>
.container {
}
.user-container {
  height: 460rpx;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  /* background-color: red; */
}
.user-container .img-container {
  position: relative;
  width: 300rpx;
  height: 300rpx;
  border-radius: 6rpx;
  border: 1rpx solid tomato;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 50rpx;
  overflow: hidden;
}
.user-container .img-container image {
  width: 100%;
  height: 100%;
  /* background: red; */
}
.user-container .img-container .cover-img {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}
.user-container .img-container .canvas {
  position: absolute;
  top: -999999rpx;
  left: -999999rpx;
  width: 100%;
  height: 100%;
}
.btn-group {
  display: flex;
  justify-content: center;
  align-items: center;
}
.btn-group button {
  margin: 0 20rpx;
}
.scroll-container {
  height: calc(100vh - 460rpx);
}
.list-container {
  display: flex;
  flex-wrap: wrap;
}
.list-container .list-item {
  width: 50%;
  height: 260rpx;
  display: flex;
  justify-content: center;
  align-items: center;
}
.list-container .list-item .img-container {
  position: relative;
  width: 200rpx;
  height: 200rpx;
}
.list-container .list-item .img-container .img {
  width: 100%;
  height: 100%;
  z-index: 1;
}
.list-container .list-item .img-container .check {
  position: absolute;
  bottom: 40rpx;
  right: 40rpx;
  z-index: 1;
  width: 36rpx;
  height: 36rpx;
  z-index: 1;
}
</style>
