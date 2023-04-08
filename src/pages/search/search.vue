<template>
	<tm-app ref="app">
		<tm-sticky>
			<template v-slot:sticky>
				<tm-sheet :margin="[0]">
					<!--  #ifdef  APP-VUE -->
					<view :style="{height:sysinfo.statusBarHeight+'px'}"></view>
					<!--  #endif -->
					<tm-input prefix="tmicon-search" searchLabel="搜索" placeholder="搜索热点" v-model="state.keyword"
						@search="onSearch"></tm-input>
				</tm-sheet>
			</template>
			<tm-result v-if="state.list.length==0" :showBtn="false" title="内容空空的" subTitle="赶紧搜索吧~"></tm-result>
			<tm-sheet :margin="[24,24,24,0]" :round="3" :shadow="2" :padding="[10]" :height="240"
				_class="flex flex-row flex-row-center-start" v-for="(item,index) in state.list" :key="index">
				<view class="flex flex-row  flex-1" @click="toDetail(item)">
					<tm-image :height="240" :width="180" :src="item.cover"></tm-image>
					<view style="display: flex;flex-direction: column;margin-left: 10rpx;">
						<tm-text :label="item.title" style="font-weight: 600;"></tm-text>
						<tm-text :label="state.type=='video'?item.director:item.author" color="grey"
							class="mb-15"></tm-text>
						<tm-text :label="item.descs" _class="text-overflow-4"></tm-text>
					</view>
				</view>
			</tm-sheet>
		</tm-sticky>
	</tm-app>
</template>

<script lang="ts" setup>
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmSheet from "@/tmui/components/tm-sheet/tm-sheet.vue"
	import tmText from "@/tmui/components/tm-text/tm-text.vue"
	import tmInput from "@/tmui/components/tm-input/tm-input.vue"
	import tmResult from "@/tmui/components/tm-result/tm-result.vue"
	import tmSticky from "@/tmui/components/tm-sticky/tm-sticky.vue"
	import { onLoad } from "@dcloudio/uni-app";
	import {
		ref,
		getCurrentInstance,
		reactive,
	} from "vue"

	const sysinfo = uni.$tm.u.getWindow()
	const { proxy } = getCurrentInstance();
	const state = reactive({
		type: 'fiction',
		keyword: '',
		list: [],
		current: 1,
	});
	onLoad((option) => {
		state.type = option?.type;
	})
	const getFiction = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/fiction/search/title/${state.keyword}`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				state.list = data;
			} else {
				proxy.tm.u.toast(msg);
				setTimeout(getFiction, 1000);
				// msg.value?.show({ model: 'warn', text: msg });
			}
			uni.hideLoading()
		});
	}
	const getComic = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/comic/search/title/${state.keyword}`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				state.list = data;
			} else {
				proxy.tm.u.toast(msg);
				setTimeout(getComic, 1000);
				// msg.value?.show({ model: 'warn', text: msg });
			}
			uni.hideLoading()
		});
	}
	const getVideo = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/video/search/title/${state.keyword}`, {}).then((res : any) => {
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
	const toDetail = (item : object) => {
		if (state.type == "fiction") {
			uni.navigateTo({
				url: `/pages/fiction_detail/fiction_detail?id=${item.fictionId}&title=${item.title}`
			});
		} else if (state.type == "comic") {
			uni.navigateTo({
				url: `/pages/comic_detail/comic_detail?id=${item.comicId}&title=${item.title}`
			});
		} else {
			uni.navigateTo({
				url: `/pages/video_detail/video_detail?id=${item.videoId}&title=${item.title}`
			});
		}
	}
	const onSearch = () => {
		if (!state.keyword) {
			return;
		}
		uni.showLoading({
			title: '搜索中'
		});
		if (state.type == "fiction") {
			getFiction();
		} else if (state.type == "comic") {
			getComic();
		} else {
			getVideo();
		}
	}
</script>

<style>

</style>