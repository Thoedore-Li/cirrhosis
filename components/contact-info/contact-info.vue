<template>
  <view class="contact-section">
    <view class="contact-title">如需专业指导，可联系：</view>
    <view class="contact-list">
      <view class="contact-item" @tap="makeCall('023-46331306')">
        <text class="department">1. 住院部：</text>
        <text class="phone">(023) 46331306</text>
      </view>
      <view class="contact-item" @tap="makeCall('023-46331315')">
        <text class="department">2. 肝病门诊：</text>
        <text class="phone">(023) 46331315</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    makeCall(phoneNumber) {
      const cleanNumber = phoneNumber.replace(/[^0-9]/g, '')
      
      // #ifdef MP-WEIXIN
      uni.makePhoneCall({
        phoneNumber: cleanNumber,
        fail() {
          uni.showModal({
            title: '提示',
            content: `请手动拨打电话：${phoneNumber}`,
            icon: 'none',
            showCancel: false
          })
        }
      })
      // #endif
      
      // #ifdef H5
      window.location.href = `tel:${cleanNumber}`
      // #endif
    }
  }
}
</script>

<style>
.contact-section {
  background: #fff;
  padding: 30rpx;
  margin: 30rpx;
  border-radius: 16rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.contact-title {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 20rpx;
}

.contact-list {
  display: flex;
  flex-direction: column;
  gap: 16rpx;
}

.contact-item {
  display: flex;
  align-items: center;
  padding: 10rpx 0;
  cursor: pointer;
  transition: opacity 0.2s;
}

.contact-item:active {
  opacity: 0.7;
}

.department {
  font-size: 28rpx;
  color: #333;
}

.phone {
  font-size: 28rpx;
  color: #07c160;
  text-decoration: underline;
  margin-left: 10rpx;
}

.contact-item::after {
  content: '';
  display: inline-block;
  width: 32rpx;
  height: 32rpx;
  margin-left: 10rpx;
  background: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0iIzA3YzE2MCI+PHBhdGggZD0iTTYuNjIgMTAuNzljLjM1IDMuMDQgMS45NSA1LjggNC4zMiA3LjU5bDIuMDMtMi4wM2MuMjUtLjI1LjYxLS4zNS45MS0uMjMgMS4wOS4zNiAyLjI1LjU1IDMuNDUuNTUuNDEgMCAuNzUuMzQuNzUuNzVWMjBjMCAuNDEtLjM0Ljc1LS43NS43NUM3LjI2IDIwLjc1IDIgMTUuNDkgMiA5YzAtLjQxLjM0LS43NS43NS0uNzVoMi45MmMuNDEgMCAuNzUuMzQuNzUuNzUgMCAxLjIuMTkgMi4zNi41NSAzLjQ1LjEyLjMuMDIuNjYtLjIzLjkxTDQuNjIgMTUuNzl6Ii8+PC9zdmc+') no-repeat center;
  background-size: contain;
}
</style> 