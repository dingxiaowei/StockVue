<template>
	<view class="kline-page">
		<view class="header">
			<text class="stock-name">贵州茅台 (600519)</text>
			<text class="stock-price">1868.50</text>
			<text class="stock-change positive">+32.80 (+1.79%)</text>
		</view>
		<view class="chart-wrap">
			<view id="klineChart" class="kline-chart" :prop="optionData" :change:prop="bindEcharts.bindUpdateChart"></view>
		</view>
		<view class="info-section">
			<view class="info-row">
				<view class="info-item">
					<text class="info-label">今开</text>
					<text class="info-value">1840.00</text>
				</view>
				<view class="info-item">
					<text class="info-label">最高</text>
					<text class="info-value positive">1875.00</text>
				</view>
				<view class="info-item">
					<text class="info-label">最低</text>
					<text class="info-value negative">1832.00</text>
				</view>
				<view class="info-item">
					<text class="info-label">昨收</text>
					<text class="info-value">1835.70</text>
				</view>
			</view>
			<view class="info-row">
				<view class="info-item">
					<text class="info-label">成交量</text>
					<text class="info-value">3.28万手</text>
				</view>
				<view class="info-item">
					<text class="info-label">成交额</text>
					<text class="info-value">61.2亿</text>
				</view>
				<view class="info-item">
					<text class="info-label">换手率</text>
					<text class="info-value">2.61%</text>
				</view>
				<view class="info-item">
					<text class="info-label">市盈率</text>
					<text class="info-value">33.56</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				optionData: {
					// 模拟贵州茅台 30 个交易日 K 线数据
					// 格式: [开盘, 收盘, 最低, 最高]
					dates: [
						'2026-01-05', '2026-01-06', '2026-01-07', '2026-01-08', '2026-01-09',
						'2026-01-12', '2026-01-13', '2026-01-14', '2026-01-15', '2026-01-16',
						'2026-01-19', '2026-01-20', '2026-01-21', '2026-01-22', '2026-01-23',
						'2026-01-26', '2026-01-27', '2026-01-28', '2026-01-29', '2026-01-30',
						'2026-02-02', '2026-02-03', '2026-02-04', '2026-02-05', '2026-02-06',
						'2026-02-09', '2026-02-10', '2026-02-11', '2026-02-12', '2026-02-13'
					],
					klineData: [
						[1780, 1795, 1770, 1800],
						[1795, 1788, 1780, 1802],
						[1790, 1810, 1785, 1815],
						[1810, 1805, 1795, 1818],
						[1805, 1825, 1800, 1830],
						[1825, 1818, 1810, 1832],
						[1820, 1835, 1815, 1840],
						[1835, 1842, 1828, 1848],
						[1840, 1830, 1822, 1845],
						[1830, 1850, 1825, 1855],
						[1850, 1845, 1838, 1858],
						[1845, 1860, 1840, 1865],
						[1858, 1852, 1845, 1868],
						[1852, 1870, 1848, 1878],
						[1870, 1862, 1855, 1880],
						[1865, 1855, 1848, 1872],
						[1855, 1840, 1835, 1860],
						[1842, 1850, 1836, 1858],
						[1850, 1865, 1845, 1870],
						[1865, 1858, 1850, 1872],
						[1860, 1872, 1855, 1880],
						[1872, 1868, 1860, 1878],
						[1870, 1882, 1865, 1888],
						[1880, 1875, 1868, 1890],
						[1878, 1885, 1872, 1895],
						[1885, 1878, 1870, 1892],
						[1880, 1860, 1855, 1885],
						[1862, 1875, 1858, 1880],
						[1875, 1870, 1862, 1882],
						[1840, 1868, 1832, 1875]
					],
					volumes: [
						32800, 28500, 35600, 30200, 41200,
						27800, 38500, 42300, 35100, 45600,
						31200, 39800, 33500, 44200, 36800,
						40500, 48200, 35800, 42100, 37500,
						39200, 34800, 43500, 38200, 41800,
						36500, 52300, 40100, 35800, 44600
					]
				}
			}
		},
		onLoad() {},
		methods: {}
	}
</script>

<!-- renderjs 模块：用于在视图层直接操作 DOM 加载 ECharts -->
<script module="bindEcharts" lang="renderjs">
	export default {
		data() {
			return {
				myChart: null
			}
		},
		mounted() {
			if (typeof window.echarts === 'object') {
				this.bindInitChart()
			} else {
				const script = document.createElement('script')
				script.src = 'https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js'
				script.onload = () => {
					this.bindInitChart()
				}
				document.head.appendChild(script)
			}
		},
		methods: {
			bindInitChart() {
				const chartDom = document.getElementById('klineChart')
				if (!chartDom) return
				this.myChart = echarts.init(chartDom)
				this.bindSetOption(this.$ownerInstance.$vm.optionData)
				window.addEventListener('resize', () => {
					if (this.myChart) {
						this.myChart.resize()
					}
				})
			},
			bindUpdateChart(newVal) {
				if (this.myChart && newVal) {
					this.bindSetOption(newVal)
				}
			},
			bindSetOption(data) {
				if (!data || !data.dates) return

				const upColor = '#ec0000'
				const downColor = '#00da3c'
				const upBorderColor = '#ec0000'
				const downBorderColor = '#00da3c'

				// 计算 MA 均线
				function calcMA(dayCount, klineData) {
					const result = []
					for (let i = 0; i < klineData.length; i++) {
						if (i < dayCount - 1) {
							result.push('-')
							continue
						}
						let sum = 0
						for (let j = 0; j < dayCount; j++) {
							sum += klineData[i - j][1] // 收盘价
						}
						result.push((sum / dayCount).toFixed(2))
					}
					return result
				}

				// 判断涨跌来给成交量柱子上色
				const volumeColors = data.klineData.map(function(item) {
					return item[1] >= item[0] ? upColor : downColor
				})

				const option = {
					backgroundColor: '#1a1a2e',
					animation: true,
					tooltip: {
						trigger: 'axis',
						axisPointer: {
							type: 'cross',
							crossStyle: { color: '#555' }
						},
						backgroundColor: 'rgba(30,30,50,0.9)',
						borderColor: '#444',
						textStyle: { color: '#eee', fontSize: 12 }
					},
					legend: {
						data: ['K线', 'MA5', 'MA10', 'MA20'],
						top: 8,
						textStyle: { color: '#ccc', fontSize: 11 },
						itemWidth: 14,
						itemHeight: 10
					},
					axisPointer: {
						link: [{ xAxisIndex: 'all' }]
					},
					grid: [
						{ left: '10%', right: '4%', top: '12%', height: '55%' },
						{ left: '10%', right: '4%', top: '74%', height: '18%' }
					],
					xAxis: [
						{
							type: 'category',
							data: data.dates,
							boundaryGap: true,
							axisLine: { lineStyle: { color: '#444' } },
							axisLabel: {
								color: '#999',
								fontSize: 10,
								rotate: 45,
								formatter: function(value) {
									return value.substring(5)
								}
							},
							splitLine: { show: false },
							min: 'dataMin',
							max: 'dataMax'
						},
						{
							type: 'category',
							gridIndex: 1,
							data: data.dates,
							boundaryGap: true,
							axisLine: { lineStyle: { color: '#444' } },
							axisLabel: { show: false },
							splitLine: { show: false },
							min: 'dataMin',
							max: 'dataMax'
						}
					],
					yAxis: [
						{
							scale: true,
							splitArea: {
								show: true,
								areaStyle: {
									color: ['rgba(255,255,255,0.02)', 'rgba(255,255,255,0.05)']
								}
							},
							axisLine: { lineStyle: { color: '#444' } },
							axisLabel: { color: '#999', fontSize: 10 },
							splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } }
						},
						{
							scale: true,
							gridIndex: 1,
							splitNumber: 2,
							axisLabel: {
								color: '#999',
								fontSize: 10,
								formatter: function(value) {
									return (value / 10000).toFixed(1) + '万'
								}
							},
							axisLine: { lineStyle: { color: '#444' } },
							splitLine: { lineStyle: { color: 'rgba(255,255,255,0.05)' } }
						}
					],
					dataZoom: [
						{
							type: 'inside',
							xAxisIndex: [0, 1],
							start: 20,
							end: 100
						},
						{
							show: true,
							xAxisIndex: [0, 1],
							type: 'slider',
							bottom: '2%',
							height: 16,
							start: 20,
							end: 100,
							borderColor: '#444',
							textStyle: { color: '#999' },
							handleStyle: { color: '#6a6aff' },
							fillerColor: 'rgba(100,100,200,0.2)'
						}
					],
					series: [
						{
							name: 'K线',
							type: 'candlestick',
							data: data.klineData,
							itemStyle: {
								color: upColor,
								color0: downColor,
								borderColor: upBorderColor,
								borderColor0: downBorderColor
							}
						},
						{
							name: 'MA5',
							type: 'line',
							data: calcMA(5, data.klineData),
							smooth: true,
							lineStyle: { width: 1, opacity: 0.8 },
							symbol: 'none',
							itemStyle: { color: '#e6c84f' }
						},
						{
							name: 'MA10',
							type: 'line',
							data: calcMA(10, data.klineData),
							smooth: true,
							lineStyle: { width: 1, opacity: 0.8 },
							symbol: 'none',
							itemStyle: { color: '#39afe6' }
						},
						{
							name: 'MA20',
							type: 'line',
							data: calcMA(20, data.klineData),
							smooth: true,
							lineStyle: { width: 1, opacity: 0.8 },
							symbol: 'none',
							itemStyle: { color: '#da6ee8' }
						},
						{
							name: '成交量',
							type: 'bar',
							xAxisIndex: 1,
							yAxisIndex: 1,
							data: data.volumes,
							itemStyle: {
								color: function(params) {
									return volumeColors[params.dataIndex]
								}
							}
						}
					]
				}

				this.myChart.setOption(option)
			}
		}
	}
</script>

<style>
	.kline-page {
		background-color: #0f0f23;
		min-height: 100vh;
		padding-bottom: 30rpx;
	}

	.header {
		padding: 30rpx;
		display: flex;
		flex-direction: column;
		align-items: flex-start;
	}

	.stock-name {
		font-size: 34rpx;
		color: #ffffff;
		font-weight: bold;
	}

	.stock-price {
		font-size: 56rpx;
		color: #ec0000;
		font-weight: bold;
		margin-top: 10rpx;
	}

	.stock-change {
		font-size: 28rpx;
		margin-top: 6rpx;
	}

	.stock-change.positive {
		color: #ec0000;
	}

	.stock-change.negative {
		color: #00da3c;
	}

	.chart-wrap {
		width: 100%;
		padding: 0 10rpx;
		box-sizing: border-box;
	}

	.kline-chart {
		width: 100%;
		height: 500px;
	}

	.info-section {
		margin: 20rpx 30rpx 0;
		background-color: rgba(255, 255, 255, 0.05);
		border-radius: 16rpx;
		padding: 24rpx;
	}

	.info-row {
		display: flex;
		justify-content: space-between;
		margin-bottom: 20rpx;
	}

	.info-row:last-child {
		margin-bottom: 0;
	}

	.info-item {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.info-label {
		font-size: 22rpx;
		color: #888;
	}

	.info-value {
		font-size: 26rpx;
		color: #ddd;
		margin-top: 6rpx;
		font-weight: 500;
	}

	.info-value.positive {
		color: #ec0000;
	}

	.info-value.negative {
		color: #00da3c;
	}
</style>
