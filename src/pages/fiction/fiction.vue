<template>
	<tm-app ref="app">
		<tm-navbar title="小说" hideHome>
			<template v-slot:right>
				<tm-icon _class="px-48" name="tmicon-search" @click="toSearch"></tm-icon>
			</template>
		</tm-navbar>
		<tm-message ref="msg"></tm-message>
		<tm-sheet :margin="[24]" :round="3" :shadow="2">
			<view style="display: flex;justify-content: space-between;">
				<view style="width: 200px;display: flex;">
					<tm-sheet :margin="[0,0,10,0]" :padding="[0]">
						<tm-text label="热门小说" style="font-weight: 600;">
						</tm-text>
					</tm-sheet>
					<tm-icon color="red" :fontSize="40" name="tmicon-HOT-copy"></tm-icon>
				</view>
				<view style="width: auto;display: flex;align-items: center;">
					<tm-sheet :margin="[0,0,10,0]" :padding="[0]" @click="msg.show({model:'load'})">
						<tm-text label="查看更多">
						</tm-text>
					</tm-sheet>
					<tm-icon name="tmicon-angle-right" :fontSize="26"></tm-icon>
				</view>
			</view>
			<tm-divider :margin="[0,10]" :padding="[0]"></tm-divider>
			<tm-skeleton v-if="!state.isReady" :margin="[24]" :padding="[24]" :height="305/2" :rows="2"
				model="rect"></tm-skeleton>
			<tm-scrollx v-else :height="610">
				<tm-grid :col="5" :width="sysinfo.width*2+200">
					<tm-grid-item v-for="(item,index) in state.carouselList" :key="index" :height="300"
						@click="toDetail(item)">
						<tm-image :height="240" :width="180" :src="item.cover"></tm-image>
						<view style="white-space: normal;">
							<tm-text _class="text-overflow-1 mt-10" :font-size="24" :label="item.title"></tm-text>
						</view>
					</tm-grid-item>
				</tm-grid>
			</tm-scrollx>
		</tm-sheet>
		<view style="width: 200px;display: flex;">
			<tm-sheet :margin="[24,0,10,0]" :padding="[0]" transprent>
				<tm-text label="最近更新" style="font-weight: 600;">
				</tm-text>
			</tm-sheet>
			<tm-icon color="red" :fontSize="40" name="tmicon-NEW-copy"></tm-icon>
		</view>
		<tm-sheet :margin="[24,0,24,24]" :padding="[0,0]" transprent>
			<tm-skeleton v-if="!state.isReady2" :margin="[0]" :padding="[0]" :height="305/2" :rows="2"
				model="line"></tm-skeleton>
			<tm-sheet :margin="[0,12]" :round="3" :shadow="2" :padding="[10]" :height="240"
				_class="flex flex-row flex-row-center-start" v-for="(item,index) in state.fictionList" :key="index">
				<view class="flex flex-row  flex-1" @click="toDetail(item)">
					<tm-image :height="240" :width="180" :src="item.cover"></tm-image>
					<view style="display: flex;flex-direction: column;margin-left: 10rpx;">
						<tm-text :label="item.title" style="font-weight: 600;"></tm-text>
						<tm-text :label="item.author" color="grey" class="mb-15"></tm-text>
						<tm-text :label="item.descs" _class="text-overflow-4"></tm-text>
					</view>
				</view>
			</tm-sheet>
		</tm-sheet>

	</tm-app>
</template>

<script lang="ts" setup>
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmSheet from "@/tmui/components/tm-sheet/tm-sheet.vue"
	import tmText from "@/tmui/components/tm-text/tm-text.vue"
	import tmIcon from "@/tmui/components/tm-icon/tm-icon.vue"
	import tmScrollx from "../../tmui/components/tm-scrollx/tm-scrollx.vue";
	import tmImage from "../../tmui/components/tm-image/tm-image.vue"
	import tmSkeleton from "@/tmui/components/tm-skeleton/tm-skeleton.vue"
	import tmDivider from "@/tmui/components/tm-divider/tm-divider.vue"
	import tmMessage from "@/tmui/components/tm-message/tm-message.vue"
	import {
		ref,
		getCurrentInstance,
		computed,
		inject,
		reactive,
	} from "vue"

	const sysinfo = uni.$tm.u.getWindow()
	const { proxy } = getCurrentInstance();
	const app = ref<InstanceType<typeof tmApp> | null>(null)
	const msg = ref<InstanceType<typeof tmMessage> | null>(null)
	const state = reactive({
		carouselList: [],
		fictionList: [],
		isReady: false,
		isReady2: false,
		current: 1,
	});
	// #ifdef H5
	const carouselWidthComputed = computed(() => {
		console.log(sysinfo.width)
		return sysinfo.width * 2 - 96;
	});
	const listWidthComputed = computed(() => {
		console.log(sysinfo.width)
		return sysinfo.width * 2 - 48 - 24;
	});
	// #endif
	// #ifdef APP-VUE
	const carouselWidthComputed = computed(() => {
		console.log(sysinfo.width)
		return sysinfo.width * 2 - 96 - 48;
	});
	const listWidthComputed = computed(() => {
		console.log(sysinfo.width)
		return sysinfo.width * 2 - 96 - 10;
	});
	// #endif
	const getCarouselData = () => {
		proxy.tm.fetch.get("https://api.pingcc.cn/fiction/search/fictionType/玄幻/1/10", {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				state.carouselList = data;
				state.isReady = true;
			} else {
				proxy.tm.u.toast(msg)
				state.isReady = false;
				setTimeout(getCarouselData, 1000);
			}
		});
	}
	const getListData = (type = "default") => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/fiction/search/fictionType/武侠/${state.current}/10`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				if (type == "default") {
					state.fictionList = []
					state.fictionList = data;
				} else {
					state.fictionList.push(...data);
					state.current = state.current + 1;
				}
				state.isReady2 = true;
			} else {
				state.isReady2 = false;
				proxy.tm.u.toast(msg)
				// msg.value?.show({ model: 'warn', text: msg })
				setTimeout(getListData, 1000);
			}

		});
	}
	const toDetail = (item : object) => {
		uni.navigateTo({
			url: `/pages/fiction_detail/fiction_detail?id=${item.fictionId}&title=${item.title}`
		});
	}
	const toSearch = () => {
		uni.navigateTo({
			url: `/pages/search/search?type=fiction`
		});
	}
	const toMore = () => { }
	uni.showLoading({
		title: '加载中'
	});
	getCarouselData()
	getListData()
	uni.hideLoading()
</script>

<style>

</style>