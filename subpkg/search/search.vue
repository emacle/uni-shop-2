<template>
	<view class="search-box">
		<!-- 使用 uni-ui 提供的搜索组件 -->
		<uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
		<!-- 搜索建议列表 -->
		<view class="sugg-list">
			<view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item.goods_id)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				show: true,
				showSync: true,
				searchVal: "",
				// 延时器的 timerId
				timer: null,
				// 搜索关键词
				kw: '',
				// 搜索结果列表
				searchResults: []
			};
		},
		methods: {
			input(e) {
				// 清除 timer 对应的延时器
				clearTimeout(this.timer)
				// 重新启动一个延时器，并把 timerId 赋值给 this.timer
				this.timer = setTimeout(() => {
					// 如果 500 毫秒内，没有触发新的输入事件，则为搜索关键词赋值
					this.kw = e
					console.log(this.kw)
					// 根据关键词，查询搜索建议列表
					this.getSearchList()
				}, 500)
			},
			// 根据搜索关键词，搜索商品建议列表
			async getSearchList() {
				console.log('getSearchList', this.kw)

				// 判断关键词是否为空
				if (this.kw === '') {
					this.searchResults = []
					return
				}
				// 发起请求，获取搜索建议列表
				const {
					data: res
				} = await uni.$http.get('/api/public/v1/goods/qsearch', {
					query: this.kw
				})
				if (res.meta.status !== 200) return uni.$showMsg()
				this.searchResults = res.message
			},
			gotoDetail(goods_id) {
				uni.navigateTo({
					// 指定详情页面的 URL 地址，并传递 goods_id 参数
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
				})
			}
		}
	}
</script>

<style lang="scss">
	.uni-searchbar {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
		position: relative;
		padding: 16rpx;
		/* 将默认的 #FFFFFF 改为 #C00000 */
		background-color: #c00000;
	}

	.search-box {
		position: sticky;
		top: 0;
		z-index: 999;
	}

	.sugg-list {
		padding: 0 5px;

		.sugg-item {
			font-size: 12px;
			padding: 13px 0;
			border-bottom: 1px solid #efefef;
			display: flex;
			align-items: center;
			justify-content: space-between;

			.goods-name {
				// 文字不允许换行（单行文本）
				white-space: nowrap;
				// 溢出部分隐藏
				overflow: hidden;
				// 文本溢出后，使用 ... 代替
				text-overflow: ellipsis;
				margin-right: 3px;
			}
		}
	}
</style>