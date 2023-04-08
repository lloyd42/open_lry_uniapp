<template>
	<tm-app ref="app">
		<tm-navbar title="漫画" hideHome>
		</tm-navbar>
		<view class="flex flex-center">
			<tm-waterfall ref="wall" :width="718">
				<tm-waterfall-item :img="item.cover" v-for="item in state.list" @click="toDetail(item)">
					<view class="pt-12 pb-12 px-12">
						<tm-text :label="item.title" _class="text-overflow-1"></tm-text>
						<tm-text color="grey" :label='item.author'></tm-text>
						<view class="flex flex-row flex-row-center-start">
							<template v-for="i in item.comicType">
								<tm-tag color="green" size="xs" :label="i"></tm-tag>
							</template>
						</view>
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

	const getComic = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/comic/search/comicType/恋爱`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				data.map((item : object) => {
					item.comicType = splitStr(item.comicType)
				})
				state.list = data;
			} else {
				proxy.tm.u.toast(msg);
				setTimeout(getComic, 1000);
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
	const splitStr = (str : string) => {
		let arr = []
		for (var i = 0; i < str.length / 2; i++) {
			arr.push(str.substring(i * 2, (i + 1) * 2))
		}
		return arr;
	}
	const toSearch = () => {
		uni.navigateTo({
			url: `/pages/search/search?type=comic`
		});
	}
	const toDetail = (item : object) => {
		uni.navigateTo({
			url: `/pages/comic_detail/comic_detail?id=${item.comicId}&title=${item.title}`
		});
	}
	uni.showLoading({
		title: '加载中'
	});
	getComic();
</script>

<style>

</style>