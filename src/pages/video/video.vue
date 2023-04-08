<template>
	<tm-app ref="app">
		<tm-navbar title="视频" hideHome>
		</tm-navbar>
		<view class="flex flex-center">
			<tm-waterfall ref="wall" :width="718">
				<tm-waterfall-item :img="item.cover" v-for="item in state.list" @click="toDetail(item)">
					<view class="flex flex-col flex-col-center-center pa-12" :style="{width:sysinfo.width-48+'rpx'}">
						<view>
							<tm-text :label="item.title" _class="text-overflow-1" :fontSize="32"></tm-text>
						</view>
						<tm-text color="grey" :label='item.director'></tm-text>
						<tm-text :label="'上映日期: '+item.releaseTime"></tm-text>
					</view>
				</tm-waterfall-item>
			</tm-waterfall>
		</view>
		<tm-float-button position="br" :btn="{ icon: 'tmicon-search',linear:'right',color:'primary' }"
			:offset="[30,150]" @click="toSearch"></tm-float-button>
		<tm-float-button v-if="state.isCanToTop" :disabledScrollTo="false" position="br"
			:btn="{ icon: 'tmicon-angle-up',linear:'',color:'white',label:'置顶' }"></tm-float-button>
	</tm-app>
</template>

<script lang="ts" setup>
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmWaterfall from "@/tmui/components/tm-waterfall/tm-waterfall.vue"
	import tmWaterfallItem from "@/tmui/components/tm-waterfall-item/tm-waterfall-item.vue"
	import {
		ref,
		getCurrentInstance,
		reactive,
	} from "vue"
	import { onPageScroll } from "@dcloudio/uni-app";

	const sysinfo = uni.$tm.u.getWindow()
	const { proxy } = getCurrentInstance();
	const state = reactive({
		list: [],
		current: 1,
		isCanToTop: false,
	});

	const getVideo = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/video/search/videoType/科幻`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				state.list = data;
			} else {
				proxy.tm.u.toast(msg);
				setTimeout(getVideo, 1000);
				// msg.value?.show({ model: 'warn', text: msg });
			}
			uni.hideLoading()
		});
	}
	onPageScroll((e) => {
		if (e.scrollTop > (sysinfo.height / 2)) {
			state.isCanToTop = true;
		} else {
			state.isCanToTop = false;
		}
	});
	const toSearch = () => {
		uni.navigateTo({
			url: `/pages/search/search?type=video`
		});
	}
	const toDetail = (item : object) => {
		uni.navigateTo({
			url: `/pages/video_detail/video_detail?id=${item.videoId}&title=${item.title}`
		});
	}
	uni.showLoading({
		title: '加载中'
	});
	getVideo();
</script>

<style>

</style>