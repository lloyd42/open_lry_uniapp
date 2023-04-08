<template>
	<tm-app ref="app">
		<tm-navbar :title="state.title" hideHome>
		</tm-navbar>
		<tm-skeleton v-if="!state.isReady" :height="305/2" :rows="2" model="line"></tm-skeleton>
		<template v-else>
			<tm-sheet :margin="[24]" :round="3" :shadow="2">
				<view class="flex flex-row mb-24">
					<tm-image :height="240" :width="180" :src="state.video.cover"></tm-image>
					<view class="flex-1 flex flex-col ml-10">
						<tm-text :label="state.video.title" :fontSize="30">
						</tm-text>
						<tm-text :label="state.video.director" class="my-10" color="grey">
						</tm-text>
						<tm-text :label="state.video.actor" _class="mb-10 text-overflow-2" color="grey">
						</tm-text>
						<view class="flex flex-row flex-row-center-start">
							<template v-for="i in state.video.videoType">
								<tm-tag color="green" size="xs" :label="i"></tm-tag>
							</template>
						</view>
					</view>
				</view>
				<tm-text :label="state.video.descs">
				</tm-text>
			</tm-sheet>
			<view class="flex flex-center">
				<tm-sheet :margin="[24,0,10,0]" :padding="[0]" transprent>
					<tm-text :label="'更新时间: ' + state.video.updateTime" color="grey">
					</tm-text>
				</tm-sheet>
			</view>
			<tm-sheet :round="3" :shadow="2" _class="pb-24">
				<tm-grid :width="sysinfo.width*2-96-48" :col="2" transprent>
					<tm-grid-item @click="toChapter(item,index)" v-for="(item,index) in state.moreData" :key="index">
						<tm-sheet :margin="[16]" :width="(sysinfo.width*2-96-48)/2-96" :round="3" :shadow="2">
							<tm-text :label="item.title" _class="text-overflow-1"></tm-text>
						</tm-sheet>
					</tm-grid-item>
				</tm-grid>
				<view v-if="showMore" class="flex flex-center" @click="addChapter">
					<tm-sheet transprent>
						<tm-text label="点击加载更多">
						</tm-text>
					</tm-sheet>
				</view>
			</tm-sheet>
		</template>
		<tm-float-button v-if="state.current>1" :disabledScrollTo="false" position="br"
			:btn="{ icon: 'tmicon-angle-up',linear:'',color:'white',label:'置顶' }" :offset="[50,150]"></tm-float-button>
		<tm-message ref="msg"></tm-message>
	</tm-app>
</template>

<script lang="ts" setup>
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmText from "@/tmui/components/tm-text/tm-text.vue"
	import tmMessage from "@/tmui/components/tm-message/tm-message.vue"
	import tmSheet from "@/tmui/components/tm-sheet/tm-sheet.vue"
	import tmFloatButton from "@/tmui/components/tm-float-button/tm-float-button.vue"
	import { onLoad, onUnload } from "@dcloudio/uni-app";
	import { useTmpiniaStore } from '@/tmui/tool/lib/tmpinia';
	import {
		ref,
		getCurrentInstance,
		computed,
		reactive,
	} from "vue"

	const sysinfo = uni.$tm.u.getWindow()
	const { proxy } = getCurrentInstance();
	const store = useTmpiniaStore();
	const app = ref<InstanceType<typeof tmApp> | null>(null)
	const msg = ref<InstanceType<typeof tmMessage> | null>(null)
	const state = reactive({
		video: {},
		id: null,
		title: '',
		moreData: [],
		current: 1,
		isReady: false,
		isOpen: false,
	});
	onLoad((option) => {
		state.id = option?.id;
		state.title = option?.title;
		uni.showLoading({
			title: '加载中'
		});
		getVideo();
	})
	const getVideo = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/videoChapter/search/${state.id}`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				data.videoType = data.videoType.split(",").filter(s => s && s.trim());
				state.video = data;
				store.tmStore.videoChapterList = data.chapterList;
				if (data.chapterList.length < 10) {
					state.moreData = data.chapterList
				} else {
					state.moreData = data.chapterList.slice(0, 10);
				}
				state.isReady = true;
			} else {
				proxy.tm.u.toast(msg);
				state.isReady = false;
				setTimeout(getVideo, 1000);
				// msg.value?.show({ model: 'warn', text: msg });
			}
			uni.hideLoading();
		});
	}
	const toChapter = (item, index) => {
		uni.navigateTo({
			url: `/pages/video_chapter/video_chapter?&title=${item.title}&src=${item.chapterPath}&desc=${state.video.descs}`
		});
	}
	const onMoreChange = () => {
		state.isOpen = !state.isOpen
	}
	const addChapter = () => {
		const start = state.current * 10;
		const end = (state.current + 1) * 10;
		const more = state.video.chapterList.slice(start, end);
		state.moreData.push(...more);
		state.current = state.current + 1;
	}
	const closeMore = () => {
		state.isOpen = false
	}
	onUnload(() => {
		store.tmStore.videoChapterList = []
	})
	const showMore = computed(() => {
		return state.video.chapterList.length > 10 && state.video.chapterList.length > state.moreData.length
	})
</script>

<style>

</style>