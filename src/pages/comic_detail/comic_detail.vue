<template>
	<tm-app ref="app">
		<tm-navbar :title="state.title" hideHome>
		</tm-navbar>
		<tm-skeleton v-if="!state.isReady" :height="305/2" :rows="2" model="line"></tm-skeleton>
		<template v-else>
			<tm-sheet :margin="[24]" :round="3" :shadow="2">
				<view class="flex flex-row mb-24">
					<tm-image :height="240" :width="180" :src="state.comic.cover"></tm-image>
					<view class="flex flex-col ml-10">
						<tm-text :label="state.comic.title" :fontSize="30">
						</tm-text>
						<tm-text :label="state.comic.author" class="mb-10" color="grey">
						</tm-text>
						<view class="flex flex-row flex-row-center-start">
							<template v-for="i in state.comic.comicType">
								<tm-tag color="green" size="xs" :label="i"></tm-tag>
							</template>
						</view>
					</view>
				</view>
				<tm-text :label="state.comic.descs">
				</tm-text>
			</tm-sheet>
			<view class="flex flex-center">
				<tm-sheet :margin="[24,0,10,0]" :padding="[0]" transprent>
					<tm-text :label="'更新时间: ' + state.comic.updateTime" color="grey">
					</tm-text>
				</tm-sheet>
			</view>
			<tm-sheet :margin="[24]" :round="3" :shadow="2">
				<tm-grid :width="sysinfo.width*2-96-48" :col="2">
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
		comic: {},
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
		getComic();
	})
	const getComic = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/comicChapter/search/${state.id}`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				data.comicType = splitStr(data.comicType)
				state.comic = data;
				store.tmStore.comicChapterList = data.chapterList;
				if (data.chapterList.length < 10) {
					state.moreData = data.chapterList
				} else {
					state.moreData = data.chapterList.slice(0, 10);
				}
				state.isReady = true;
			} else {
				proxy.tm.u.toast(msg);
				state.isReady = false;
				setTimeout(getComic, 1000);
				// msg.value?.show({ model: 'warn', text: msg });
			}
			uni.hideLoading();
		});
	}
	const toChapter = (item, index) => {
		uni.navigateTo({
			url: `/pages/comic_chapter/comic_chapter?id=${item.chapterId}&title=${item.title}&index=${index}`
		});
	}
	const onMoreChange = () => {
		state.isOpen = !state.isOpen
	}
	const addChapter = () => {
		const start = state.current * 10;
		const end = (state.current + 1) * 10;
		const more = state.comic.chapterList.slice(start, end);
		state.moreData.push(...more);
		state.current = state.current + 1;
	}
	const splitStr = (str : string) => {
		let arr = []
		for (var i = 0; i < str.length / 2; i++) {
			arr.push(str.substring(i * 2, (i + 1) * 2))
		}
		return arr;
	}
	const closeMore = () => {
		state.isOpen = false
	}
	onUnload(() => {
		store.tmStore.comicChapterList = []
	})
	const showMore = computed(() => {
		return state.comic.chapterList.length > 10 && state.comic.chapterList.length > state.moreData.length
	})
</script>

<style>

</style>