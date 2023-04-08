<template>
	<tm-app ref="app">
		<view class="flex flex-center">
			<video id="myVideo" :src="state.src" @error="videoErrorCallback" controls
				style="width: 100vh;padding: 0;margin: 0;display: flex; align-items: center;justify-content: center;"></video>
		</view>
		<tm-sheet transprent>
			<tm-text label="简介" :fontSize="40"></tm-text>
			<tm-text :label="state.desc" :fontSize="32"></tm-text>
		</tm-sheet>
	</tm-app>
</template>

<script lang="ts" setup>
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmSheet from "@/tmui/components/tm-sheet/tm-sheet.vue"
	import tmText from "@/tmui/components/tm-text/tm-text.vue"
	import { onLoad, onReady } from "@dcloudio/uni-app";
	import {
		ref,
		reactive,
		getCurrentInstance
	} from "vue"

	const { proxy } = getCurrentInstance();
	const state = reactive({
		src: "",
		title: "",
		desc: "",
	});
	onLoad((option) => {
		state.title = option?.title;
		state.src = option?.src;
		state.desc = option?.desc;
		uni.showLoading({
			title: '加载中'
		});
	})
	const videoErrorCallback = (e : any) => {
		proxy.tm.u.toast(e.target.errMsg);
	}
	onReady(() => {
		uni.hideLoading();
	})
</script>

<style>

</style>