<template>
  <view class="hospital-header">
    <view class="header-content">
      <image class="logo" src="/static/hospital-logo.png" mode="aspectFit"></image>
      <view class="title-container">
        <text class="hospital-name">荣昌区人民医院</text>
        <text class="department-name">应急中心及肝病中心</text>
      </view>
    </view>
  </view>
</template>

<style>
.hospital-header {
  background-color: #fff;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.1);
  padding: 20rpx 0;
}

.header-content {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 30rpx;
}

.logo {
  width: 80rpx;
  height: 80rpx;
  margin-right: 20rpx;
}

.title-container {
  display: flex;
  flex-direction: column;
}

.hospital-name {
  font-size: 32rpx;
  color: #07c160;
  font-weight: bold;
}

.department-name {
  font-size: 24rpx;
  color: #666;
  margin-top: 4rpx;
}
</style> 