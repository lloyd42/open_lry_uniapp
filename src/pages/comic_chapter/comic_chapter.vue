<template>
	<tm-app ref="app">
		<tm-navbar :title="state.title" hideHome>
		</tm-navbar>
		<tm-skeleton v-if="!state.isReady" :height="305/2" :rows="2" model="line"></tm-skeleton>
		<template v-else>
			<view @click="state.showMenu=!state.showMenu">
				<tm-image v-for="item in state.chapter" :width="sysinfo.width*2" :padding="[0]" :margin="[0]"
					:height="(sysinfo.height-sysinfo.statusBarHeight-44)*2" :src="item" model="widthFix">
				</tm-image>
			</view>
		</template>
		<tm-translate ref="translate" name="up" v-show="state.showMenu">
			<tm-tabbar :autoSelect="false">
				<tm-tabbar-item @click="openChapterList" text="目录" icon='tmicon-outdent'></tm-tabbar-item>
				<tm-tabbar-item @click="openProgress" text="进度" icon='tmicon-map-fill
				'></tm-tabbar-item>
				<!-- <tm-tabbar-item @click="openSetting" text="设置" icon='tmicon-resource'></tm-tabbar-item> -->

				<tm-tabbar-item @click="switchDark" :text="isDark?'白天':'夜晚'"
					:icon="isDark?'tmicon-md-moon':'tmicon-ios-sunny'"></tm-tabbar-item>
			</tm-tabbar>
		</tm-translate>
		<tm-drawer ref="drawer" placement="left" hideHeader v-model:show="state.showDrawer">
			<tm-sheet transprent @touchmove.stop.prevent="return" :padding="[0]" :margin="[0]">
				<tm-text label="目录" :fontSize="40" style="font-weight: 600;" _class="mt-24">
				</tm-text>
				<tm-divider></tm-divider>
				<tm-virtual-list :firstLoad="false" :width="500"
					:height="sysinfo.height*2-sysinfo.statusBarHeight*2-96-72" :data="state.chapterList"
					:itemHeight="30">
					<template v-slot:default="{data}">
						<view class="flex flex-col flex-col-center-center">
							<tm-sheet :margin="[0]" :padding="[0,20]" :border="1" borderDirection="bottom" transprent
								:width="500-96" v-for="(item,index) in data" :key="index"
								@click="changeChapter(item,index)">
								<tm-text _class="text-overflow-1" :color="state.id==item.chapterId?'grey':'black'"
									:label="item.title"></tm-text>
							</tm-sheet>
							<view class="flex flex-center my-48">
								<tm-text label="结束啦~" color="grey">
								</tm-text>
							</view>
						</view>
					</template>
				</tm-virtual-list>
			</tm-sheet>
		</tm-drawer>
		<tm-drawer ref="progressDrawer" :height="350" hideHeader placement="bottom"
			v-model:show="state.showProgressDrawer">
			<tm-sheet transprent :margin="[24,24,0,24]">
				<view class="flex flex-row flex-row-center-between">
					<tm-button :margin="[0,0,24,0]" :width="80" :height="80" :round="12" icon="tmicon-angle-left"
						:fontSize="40" transprent :disabled="disabledPrev" @click="prevChapter"
						:fontColor="disabledPrev?'grey':'black'"></tm-button>
					<tm-text :label="state.title"></tm-text>
					<tm-button :margin="[24,0,0,0]" :width="80" :height="80" :round="12" icon="tmicon-angle-right"
						:fontSize="40" transprent :disabled="disabledNext" @click="nextChapter"
						:fontColor="disabledNext?'grey':'black'"></tm-button>
				</view>
				<view class="flex flex-center">
					<tm-text :label="progress" color="grey"></tm-text>
				</view>
			</tm-sheet>
			<tm-sheet transprent :margin="[24,0,24,24]" :padding="[24,0,24,24]">
				<view class="flex flex-center">
					<slider style="width: 600rpx;" :value="state.index" @change="sliderChange3" step="1" min="0"
						:max="state.chapterList.length" />
				</view>
			</tm-sheet>
		</tm-drawer>
		<tm-message ref="msg"></tm-message>
	</tm-app>
</template>

<script lang="ts" setup>
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmMessage from "@/tmui/components/tm-message/tm-message.vue"
	import tmText from "@/tmui/components/tm-text/tm-text.vue"
	import tmSheet from "@/tmui/components/tm-sheet/tm-sheet.vue"
	import tmTranslate from "@/tmui/components/tm-translate/tm-translate.vue"
	import tmDrawer from "@/tmui/components/tm-drawer/tm-drawer.vue"
	import tmDivider from "@/tmui/components/tm-divider/tm-divider.vue"
	import tmVirtualList from "@/tmui/components/tm-virtual-list/tm-virtual-list.vue"
	import tmButton from "../../tmui/components/tm-button/tm-button.vue"
	import tmImage from "@/tmui/components/tm-image/tm-image.vue"
	import { onLoad } from "@dcloudio/uni-app";
	import { useTmpiniaStore } from '@/tmui/tool/lib/tmpinia';
	import {
		ref,
		getCurrentInstance,
		computed,
		reactive,
		onUpdated,
	} from "vue"
	import console from "console"

	const sysinfo = uni.$tm.u.getWindow()
	const { proxy } = getCurrentInstance();
	const store = useTmpiniaStore();
	const app = ref<InstanceType<typeof tmApp> | null>(null)
	const msg = ref<InstanceType<typeof tmMessage> | null>(null)
	const translate = ref<InstanceType<typeof tmTranslate> | null>(null)
	const drawer = ref<InstanceType<typeof tmDrawer> | null>(null)
	const progressDrawer = ref<InstanceType<typeof tmDrawer> | null>(null)
	const state = reactive({
		chapter: [],
		chapterList: [],
		id: "",
		title: "",
		index: 0,
		isReady: false,
		fontSize: 16,
		brightness: 0,
		theme: 'default',
		showMenu: false,
		showDrawer: false,
		showProgressDrawer: false,
	});
	onLoad((option) => {
		state.id = option?.id;
		state.title = option?.title;
		state.index = Number(option?.index);
		state.chapterList = store.tmStore.comicChapterList
		getChapter();
	});
	const getChapter = () => {
		proxy.tm.fetch.get(`https://api.pingcc.cn/comicContent/search/${state.id}`, {}).then((res : any) => {
			const { data, code, msg } = res.data;
			if (code == 0) {
				state.chapter = data;
				state.isReady = true;
			} else {
				proxy.tm.u.toast(msg);
				state.isReady = false;
				setTimeout(getChapter, 1000);
				// msg.value?.show({ model: 'warn', text: msg })
			}
		});
	}
	const changeChapter = (item : object, index : number) => {
		state.index = index;
		state.id = item.chapterId;
		state.title = item.title;
		getChapter();
		drawer.value.close();
	}
	const openChapterList = () => {
		state.showDrawer = !state.showDrawer;
	}
	const openProgress = () => {
		state.showProgressDrawer = !state.showProgressDrawer;
	}
	const switchDark = () => {
		//tmicon-ios-sunny
		// store.tmStore.dark = !store.tmStore.dark;
		app.value?.setDark();
	}
	const isDark = computed(() => {
		return store.tmStore.dark;
	})
	const sliderChange = (e) => {
		console.log(e.detail.value)
		state.fontSize = e.detail.value;
	}
	const sliderChange2 = (e) => {
		state.brightness = e.detail.value;
		uni.setScreenBrightness({
			value: e.detail.value,
			success: function () {
				console.log('success');
			}
		});
	}
	const sliderChange3 = (e) => {
		state.index = e.detail.value;
		const id = state.chapterList[state.index].chapterId;
		const title = state.chapterList[state.index].title;
		state.id = id;
		state.title = title;
		getChapter()
	}
	uni.getScreenBrightness({
		success: function (res) {
			state.brightness = res.value
		}
	});
	onUpdated(() => {
		console.log(state.bg)
	})
	const progress = computed(() => {
		return ((state.index / state.chapterList.length) * 100).toFixed(1) + "%";
	})
	const disabledNext = computed(() => {
		return state.index == state.chapterList.length;
	})
	const disabledPrev = computed(() => {
		return state.index == 0;
	})
	const prevChapter = () => {
		state.index = state.index - 1;
		const id = state.chapterList[state.index].chapterId;
		const title = state.chapterList[state.index].title;
		state.id = id;
		state.title = title;
		getChapter()
	}
	const nextChapter = () => {
		console.log(state.chapterList[state.index].chapterId)
		state.index = state.index + 1;
		const id = state.chapterList[state.index].chapterId;
		const title = state.chapterList[state.index].title;
		state.id = id;
		state.title = title;
		getChapter()
	}
</script>

<style>

</style>