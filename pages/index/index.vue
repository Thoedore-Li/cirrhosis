<template>
	<view class="container">
		<hospital-header></hospital-header>
		
		<view class="main-content">
			<!-- 历史数据入口 -->
			<view class="history-entry" @tap="showHistoryModal">
				<text class="history-text">录入历史数据</text>
				<text class="history-icon">+</text>
			</view>
			
			<!-- 体重记录卡片 -->
			<view class="card">
				<view class="card-header">
					<text class="card-title">体重记录</text>
					<text class="card-subtitle">每日测量体重，监测腹水变化</text>
				</view>
				<view class="form-item">
					<text class="label">今日体重 (kg)</text>
					<input 
						type="digit" 
						v-model="dailyData.weight" 
						placeholder="请输入今日体重"
						@input="validateInput"
						class="input-box"
					/>
				</view>
			</view>

			<!-- 尿量记录卡片 -->
			<view class="card">
				<view class="card-header">
					<text class="card-title">尿量记录</text>
					<text class="card-subtitle">记录每次排尿量，自动计算日总量</text>
				</view>
				<view class="form-item">
					<text class="label">本次尿量 (ml)</text>
					<view class="input-group">
						<input 
							type="number" 
							v-model="currentUrine" 
							placeholder="请输入本次尿量"
							@input="validateInput"
							class="input-box"
						/>
						<button class="add-btn" @click="addUrine" size="mini">添加</button>
					</view>
				</view>
				<view class="urine-summary" v-if="todayUrineList.length > 0">
					<text class="summary-title">今日记录</text>
					<view class="urine-total">
						<text>总量：{{dailyData.urine}}ml</text>
						<text>次数：{{todayUrineList.length}}次</text>
					</view>
					<view class="urine-list">
						<view class="urine-item" v-for="(item, index) in todayUrineList" :key="index">
							<text class="time">{{item.time}}</text>
							<text class="value">{{item.value}}ml</text>
						</view>
					</view>
				</view>
			</view>

			<!-- 操作按钮 -->
			<view class="action-buttons">
				<button @click="saveData" type="primary" class="save-btn">保存今日数据</button>
				<button @click="goToTrend" class="trend-btn">查看趋势分析</button>
			</view>
			<contact-info></contact-info>
		</view>

		<!-- 历史数据录入弹窗 -->
		<view class="modal" v-if="showModal" @tap="closeModal">
			<view class="modal-content" @tap.stop>
				<view class="modal-header">
					<text class="modal-title">录入历史数据</text>
					<text class="modal-close" @tap="closeModal">×</text>
				</view>
				
				<view class="modal-body">
					<view class="form-item">
						<text class="label">选择日期</text>
						<picker 
							mode="date" 
							:value="historyData.date" 
							:end="today"
							@change="onDateChange"
							class="date-picker"
						>
							<view class="picker-value">
								{{historyData.date || '请选择日期'}}
							</view>
						</picker>
					</view>
					
					<view class="form-item">
						<text class="label">体重 (kg)</text>
						<input 
							type="digit"
							v-model="historyData.weight"
							placeholder="请输入体重"
							class="input-box"
						/>
					</view>
					
					<view class="form-item">
						<text class="label">尿量 (ml)</text>
						<input 
							type="number"
							v-model="historyData.urine"
							placeholder="请输入尿量"
							class="input-box"
						/>
					</view>
				</view>
				
				<view class="modal-footer">
					<button class="cancel-btn" @tap="closeModal">取消</button>
					<button class="confirm-btn" @tap="saveHistoryData">保存</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import HospitalHeader from '@/components/hospital-header/hospital-header.vue'
	import ContactInfo from '@/components/contact-info/contact-info.vue'
	
	export default {
		components: {
			HospitalHeader,
			ContactInfo
		},
		data() {
			return {
				dailyData: {
					date: '',
					urine: '',
					weight: ''
				},
				currentUrine: '',
				todayUrineList: [],
				showModal: false,
				today: new Date().toISOString().split('T')[0],
				historyData: {
					date: '',
					weight: '',
					urine: ''
				}
			}
		},
		onLoad() {
			// 初始化今日尿量列表
			const today = new Date().toISOString().split('T')[0]
			this.todayUrineList = uni.getStorageSync(`urineList_${today}`) || []
			this.updateTotalUrine()
		},
		methods: {
			validateInput(e) {
				const value = e.detail.value
				if (value < 0) {
					uni.showToast({
						title: '请输入正数',
						icon: 'none'
					})
					return ''
				}
				return value
			},
			
			saveData() {
				if (!this.dailyData.urine || !this.dailyData.weight) {
					uni.showToast({
						title: '请填写完整数据',
						icon: 'none'
					})
					return
				}
				
				this.dailyData.date = new Date().toISOString().split('T')[0]
				
				// 获取历史数据
				let historyData = uni.getStorageSync('waterData') || []
				
				// 检查是否已存在今日数据
				const todayIndex = historyData.findIndex(item => item.date === this.dailyData.date)
				if (todayIndex !== -1) {
					historyData[todayIndex] = this.dailyData
				} else {
					historyData.push(this.dailyData)
				}
				
				// 只保留最近7天的数据
				historyData = historyData.slice(-7)
				
				uni.setStorageSync('waterData', historyData)
				
				uni.showToast({
					title: '保存成功',
					icon: 'success'
				})
				
				// 检查异常情况
				this.checkAbnormal(historyData)
			},
			
			checkAbnormal(historyData) {
				if (historyData.length < 2) return
				
				const today = historyData[historyData.length - 1]
				const yesterday = historyData[historyData.length - 2]
				
				const weightDiff = today.weight - yesterday.weight
				
				// 检查体重和尿量异常
				const hasWeightWarning = weightDiff >= 1
				const hasUrineWarning = Number(today.urine) < 500
				
				if (hasWeightWarning || hasUrineWarning) {
					let warningMsg = ''
					
					if (hasWeightWarning) {
						warningMsg += `您的体重在24小时内增加了${weightDiff.toFixed(1)}公斤\n`
					}
					if (hasUrineWarning) {
						warningMsg += `您的尿量偏少(${today.urine}ml)\n`
					}
					
					uni.showModal({
						title: '健康提醒',
						content: warningMsg + '\n建议及时就医并查看健康指导',
						showCancel: false,
						success: () => {
							uni.navigateTo({
								url: '/pages/guide/guide'
							})
						}
					})
				}
			},
			
			goToTrend() {
				uni.navigateTo({
					url: '/pages/trend/trend'
				})
			},
			addUrine() {
				if (!this.currentUrine) {
					uni.showToast({
						title: '请输入尿量',
						icon: 'none'
					})
					return
				}
				
				const urineValue = Number(this.currentUrine)
				if (urineValue <= 0) {
					uni.showToast({
						title: '请输入有效尿量',
						icon: 'none'
					})
					return
				}
				
				const today = new Date().toISOString().split('T')[0]
				this.todayUrineList.push({
					value: urineValue,
					time: new Date().toLocaleTimeString()
				})
				
				// 保存尿量列表
				uni.setStorageSync(`urineList_${today}`, this.todayUrineList)
				
				// 更新总尿量
				this.updateTotalUrine()
				
				// 清空输入框
				this.currentUrine = ''
				
				uni.showToast({
					title: '添加成功',
					icon: 'success'
				})
			},
			
			updateTotalUrine() {
				const total = this.todayUrineList.reduce((sum, item) => sum + item.value, 0)
				this.dailyData.urine = String(total)
			},
			
			showHistoryModal() {
				this.showModal = true
				this.historyData = {
					date: '',
					weight: '',
					urine: ''
				}
			},
			
			closeModal() {
				this.showModal = false
			},
			
			onDateChange(e) {
				this.historyData.date = e.detail.value
			},
			
			saveHistoryData() {
				if (!this.historyData.date || !this.historyData.weight || !this.historyData.urine) {
					uni.showToast({
						title: '请填写完整数据',
						icon: 'none'
					})
					return
				}
				
				// 获取历史数据
				let historyData = uni.getStorageSync('waterData') || []
				
				// 检查是否已存在该日期的数据
				const existingIndex = historyData.findIndex(item => item.date === this.historyData.date)
				
				if (existingIndex !== -1) {
					uni.showModal({
						title: '提示',
						content: '该日期已存在数据，是否覆盖？',
						success: (res) => {
							if (res.confirm) {
								historyData[existingIndex] = { ...this.historyData }
								this.saveAndClose(historyData)
							}
						}
					})
				} else {
					historyData.push({ ...this.historyData })
					this.saveAndClose(historyData)
				}
			},
			
			saveAndClose(historyData) {
				// 按日期排序
				historyData.sort((a, b) => new Date(a.date) - new Date(b.date))
				
				// 只保留最近7天
				historyData = historyData.slice(-7)
				
				uni.setStorageSync('waterData', historyData)
				
				uni.showToast({
					title: '保存成功',
					icon: 'success'
				})
				
				this.closeModal()
			}
		}
	}
</script>

<style>
	.container {
		min-height: 100vh;
		background-color: #f5f7fa;
	}

	.main-content {
		padding: 30rpx;
	}

	.card {
		background: #fff;
		border-radius: 16rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
	}

	.card-header {
		margin-bottom: 20rpx;
	}

	.card-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.card-subtitle {
		font-size: 24rpx;
		color: #999;
		margin-top: 4rpx;
		display: block;
	}

	.form-item {
		margin-bottom: 20rpx;
	}

	.label {
		font-size: 28rpx;
		color: #666;
		margin-bottom: 10rpx;
		display: block;
	}

	.input-box {
		height: 80rpx;
		border: 2rpx solid #e5e5e5;
		border-radius: 8rpx;
		padding: 0 20rpx;
		font-size: 28rpx;
	}

	.input-group {
		display: flex;
		align-items: center;
		gap: 20rpx;
	}

	.add-btn {
		background: #07c160;
		color: #fff;
		border-radius: 8rpx;
		height: 80rpx;
		line-height: 80rpx;
		padding: 0 40rpx;
	}

	.urine-summary {
		margin-top: 30rpx;
		border-top: 2rpx solid #f5f5f5;
		padding-top: 20rpx;
	}

	.summary-title {
		font-size: 28rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.urine-total {
		display: flex;
		justify-content: space-between;
		font-size: 28rpx;
		color: #07c160;
		margin-bottom: 20rpx;
	}

	.urine-list {
		max-height: 300rpx;
		overflow-y: auto;
	}

	.urine-item {
		display: flex;
		justify-content: space-between;
		padding: 16rpx 0;
		border-bottom: 2rpx solid #f5f5f5;
		font-size: 26rpx;
	}

	.time {
		color: #999;
	}

	.value {
		color: #333;
		font-weight: 500;
	}

	.action-buttons {
		padding: 30rpx 0;
	}

	.save-btn {
		background: #07c160;
		color: #fff;
		margin-bottom: 20rpx;
	}

	.trend-btn {
		background: #fff;
		color: #07c160;
		border: 2rpx solid #07c160;
	}

	button {
		font-size: 32rpx;
		border-radius: 8rpx;
	}

	.history-entry {
		display: flex;
		align-items: center;
		justify-content: space-between;
		background: #fff;
		padding: 20rpx 30rpx;
		margin-bottom: 30rpx;
		border-radius: 16rpx;
		box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
	}

	.history-text {
		font-size: 28rpx;
		color: #333;
	}

	.history-icon {
		width: 40rpx;
		height: 40rpx;
		line-height: 36rpx;
		text-align: center;
		border-radius: 50%;
		background: #07c160;
		color: #fff;
		font-size: 32rpx;
	}

	.modal {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(0, 0, 0, 0.5);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 999;
	}

	.modal-content {
		width: 80%;
		background: #fff;
		border-radius: 16rpx;
		overflow: hidden;
	}

	.modal-header {
		padding: 30rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-bottom: 1rpx solid #eee;
	}

	.modal-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.modal-close {
		font-size: 40rpx;
		color: #999;
		padding: 0 20rpx;
	}

	.modal-body {
		padding: 30rpx;
	}

	.date-picker {
		height: 80rpx;
		border: 2rpx solid #e5e5e5;
		border-radius: 8rpx;
		padding: 0 20rpx;
	}

	.picker-value {
		line-height: 80rpx;
		font-size: 28rpx;
		color: #333;
	}

	.modal-footer {
		padding: 20rpx;
		display: flex;
		justify-content: flex-end;
		gap: 20rpx;
		border-top: 1rpx solid #eee;
	}

	.cancel-btn, .confirm-btn {
		padding: 0 40rpx;
		height: 70rpx;
		line-height: 70rpx;
		border-radius: 8rpx;
		font-size: 28rpx;
	}

	.cancel-btn {
		background: #f5f5f5;
		color: #666;
	}

	.confirm-btn {
		background: #07c160;
		color: #fff;
	}
</style>
