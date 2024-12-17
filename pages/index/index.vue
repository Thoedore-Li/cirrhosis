<template>
	<view class="container">
		<view class="form-item">
			<text class="label">今日尿量(ml)</text>
			<input 
				type="number" 
				v-model="dailyData.urine" 
				placeholder="请输入今日尿量"
				@input="validateInput"
			/>
		</view>
		
		<view class="form-item">
			<text class="label">今日体重(kg)</text>
			<input 
				type="digit" 
				v-model="dailyData.weight" 
				placeholder="请输入今日体重"
				@input="validateInput"
			/>
		</view>

		<button @click="saveData" type="primary">保存数据</button>
		<button @click="goToTrend" type="default">查看趋势</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				dailyData: {
					date: '',
					urine: '',
					weight: ''
				}
			}
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
				
				if (weightDiff >= 1) {
					uni.showModal({
						title: '健康提醒',
						content: '您的体重较昨日增加超过1公斤，建议及时就医',
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
			}
		}
	}
</script>

<style>
	.container {
		padding: 20px;
	}
	.form-item {
		margin-bottom: 20px;
	}
	.label {
		display: block;
		margin-bottom: 5px;
	}
	input {
		width: 100%;
		height: 40px;
		border: 1px solid #ddd;
		border-radius: 4px;
		padding: 0 10px;
	}
	button {
		margin-top: 20px;
	}
</style>
