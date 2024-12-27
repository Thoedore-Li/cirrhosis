<template>
  <view class="guide-container">
    <hospital-header></hospital-header>
    <!-- 个性化提醒部分 -->
    <view class="alert-section" v-if="hasWarning">
      <view class="alert-box">
        <text class="alert-title">⚠️ 健康提醒</text>
        <text class="alert-content">{{warningMessage}}</text>
      </view>
    </view>

    <!-- 健康教育内容部分 -->
    <view class="guide-section">
      <view class="section-item" v-for="(section, index) in guideContent" :key="index">
        <view class="section-title">
          <text>{{section.title}}</text>
        </view>
        <view class="section-content">
          <view v-for="(item, idx) in section.content" :key="idx" class="content-item">
            <text>{{item}}</text>
          </view>
        </view>
      </view>
    </view>
    <contact-info></contact-info>
  </view>
</template>

<script>
import ContactInfo from '@/components/contact-info/contact-info.vue'

export default {
  data() {
    return {
      hasWarning: false,
      warningMessage: '',
      guideContent: [
        {
          title: '饮食管理',
          content: [
            '每日摄入盐分应控制在2克以内（约5克食盐）',
            '避免高盐食品，如腌制食品、罐头、泡菜等',
            '保持适量蛋白质摄入（每日1.2-1.5g/kg体重）',
            '补充足够热量（25-30kcal/kg体重）'
          ]
        },
        {
          title: '生活习惯',
          content: [
            '必须完全戒酒，避免加重肝损伤',
            '每日监测体重，若增加超过0.5公斤/天需就医',
            '适度休息，避免重体力劳动',
            '保持规律排便，预防便秘'
          ]
        },
        {
          title: '用药管理',
          content: [
            '遵医嘱服用利尿剂',
            '定期复查电解质水平',
            '避免使用可能加重肝损伤的药物'
          ]
        }
      ]
    }
  },
  onShow() {
    this.checkPatientData()
  },
  methods: {
    checkPatientData() {
      const historyData = uni.getStorageSync('waterData') || []
      if (historyData.length < 2) return
      
      const today = historyData[historyData.length - 1]
      const yesterday = historyData[historyData.length - 2]
      
      // 检查体重变化
      const weightDiff = today.weight - yesterday.weight
      const hasWeightWarning = weightDiff >= 1
      const hasUrineWarning = Number(today.urine) < 500
      
      if (hasWeightWarning || hasUrineWarning) {
        this.hasWarning = true
        let warningMsg = '警告：\n'
        
        if (hasWeightWarning) {
          warningMsg += `• 您的体重在24小时内增加了${weightDiff.toFixed(1)}公斤\n`
        }
        if (hasUrineWarning) {
          warningMsg += `• 您的尿量偏少（${today.urine}ml）\n`
        }
        
        this.warningMessage = warningMsg + '\n建议：\n' +
          '1. 立即就医复诊\n' +
          '2. 严格限制盐分摄入（每日<5g）\n' +
          '3. 遵医嘱服用利尿剂\n' +
          '4. 每日监测体重和尿量\n' +
          '5. 必要时限制液体摄入'
      }
    }
  },
  
  // 分享给好友
  onShareAppMessage() {
    return {
      title: '腹水患者健康指导-荣昌区人民医院',
      path: '/pages/guide/guide'
    }
  },
  
  // 分享到朋友圈
  onShareTimeline() {
    return {
      title: '腹水患者健康指导-荣昌区人民医院'
    }
  }
}
</script>

<style>
.guide-container {
  background-color: #f5f7fa;
  min-height: 100vh;
}

.alert-section {
  padding: 30rpx;
}

.alert-box {
  background-color: #fff3f3;
  border-radius: 8px;
  padding: 15px;
  border: 1px solid #ffcdd2;
  box-shadow: 0 2rpx 10rpx rgba(255, 205, 210, 0.2);
}

.alert-title {
  color: #f44336;
  font-weight: bold;
  font-size: 16px;
  display: block;
  margin-bottom: 10px;
}

.alert-content {
  color: #d32f2f;
  font-size: 14px;
  line-height: 1.5;
}

.guide-section {
  background-color: #ffffff;
  border-radius: 8px;
  margin: 30rpx;
  padding: 30rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
}

.section-item {
  margin-bottom: 40rpx;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #07c160;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.section-content {
  padding: 10px 0;
}

.content-item {
  padding: 8px 0;
  color: #666;
  font-size: 14px;
  line-height: 1.5;
}

.content-item:not(:last-child) {
  border-bottom: 1px solid #f5f5f5;
}
</style> 