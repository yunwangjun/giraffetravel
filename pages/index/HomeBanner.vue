<template>
  <view class="banner-container">
    <swiper 
      class="banner-swiper" 
      :indicator-dots="true"
      :autoplay="true"
      :interval="3000"
      :duration="500"
      :circular="true"
      indicator-color="rgba(255, 255, 255, 0.5)"
      indicator-active-color="#ffffff"
    >
      <swiper-item v-for="(item, index) in bannerList" :key="index">
        <view class="banner-item">
          <image 
            :src="item.img" 
            mode="aspectFill" 
            class="banner-image"
            @error="handleImageError"
          />
          <!-- 如果需要显示 text 内容，可以取消下面的注释 -->
          <!-- <view class="banner-text">{{ item.text }}</view> -->
        </view>
      </swiper-item>
    </swiper>
    
    <!-- 加载中状态 -->
    <view v-if="loading" class="loading-state">
      <text>加载中...</text>
    </view>
    
    <!-- 加载失败状态 -->
    <view v-if="!loading && bannerList.length === 0 && !hasLoaded" class="error-state">
      <text>加载失败，点击重试</text>
    </view>
  </view>
</template>

<script>
export default {
  name: 'HomeBanner',
  data() {
    return {
      bannerList: [],
      loading: false,
      hasLoaded: false,
      baseUrl: 'https://www.paris-girafe.com/api/v1/index/banner'
    }
  },
  onLoad() {
    this.fetchBannerData()
  },
  // 支持下拉刷新时重新加载
  onPullDownRefresh() {
    this.fetchBannerData().finally(() => {
      uni.stopPullDownRefresh()
    })
  },
  methods: {
    async fetchBannerData() {
      this.loading = true
      try {
        const [error, res] = await uni.request({
          url: this.baseUrl,
          method: 'GET',
          header: {
            'Content-Type': 'application/json'
          }
        })
        
        if (error) {
          console.error('请求失败:', error)
          this.hasLoaded = true
          return
        }
        
        const { data } = res
        if (data.code === 1 && data.data) {
          this.bannerList = data.data
        } else {
          console.warn('接口返回异常:', data.msg)
        }
        this.hasLoaded = true
      } catch (err) {
        console.error('网络错误:', err)
        this.hasLoaded = true
      } finally {
        this.loading = false
      }
    },
    handleImageError(e) {
      console.error('图片加载失败:', e.detail.errMsg)
    },
    /**
     * 分享到微信好友
     */
    onShareAppMessage() {
      return {
        title: '巴黎长颈鹿 - 精彩推荐',
        path: '/pages/index/index',
        imageUrl: this.bannerList.length > 0 ? this.bannerList[0].img : ''
      }
    },
    /**
     * 分享到微信朋友圈
     */
    onShareTimeline() {
      return {
        title: '巴黎长颈鹿 - 精彩推荐',
        query: '',
        imageUrl: this.bannerList.length > 0 ? this.bannerList[0].img : ''
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.banner-container {
  width: 100%;
  position: relative;
  
  .banner-swiper {
    width: 100%;
    height: 360rpx;
    
    .banner-item {
      width: 100%;
      height: 100%;
      position: relative;
      
      .banner-image {
        width: 100%;
        height: 100%;
        display: block;
      }
      
      .banner-text {
        position: absolute;
        bottom: 20rpx;
        left: 20rpx;
        color: #ffffff;
        font-size: 28rpx;
        background-color: rgba(0, 0, 0, 0.5);
        padding: 8rpx 16rpx;
        border-radius: 8rpx;
      }
    }
  }
  
  .loading-state,
  .error-state {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #f5f5f5;
    color: #999;
    font-size: 28rpx;
  }
  
  .error-state {
    cursor: pointer;
    
    &:active {
      opacity: 0.7;
    }
  }
}
</style>
