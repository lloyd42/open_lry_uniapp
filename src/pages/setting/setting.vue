<template>
	<tm-app ref="app">
		<tm-navbar title="设置" hideHome>
		</tm-navbar>
		<tm-cell :margin="[24, 16, 24, 16]" @click="setLocal" showAvatar :round="4" :titleFontSize="30"
			:title="language('index.com.setLocal')" :rightText="language('language')">
			<template v-slot:avatar>
				<tm-icon color="primary" :font-size="38" name="tmicon-resource"></tm-icon>
			</template>
		</tm-cell>
		<tm-cell :margin="[24, 0]" showAvatar :round="4" :titleFontSize="30" :title="language('index.com.autoDark')">
			<template v-slot:avatar>
				<tm-icon color="pink" :font-size="38" name="tmicon-ios-color-palette"></tm-icon>
			</template>
			<template v-slot:right>
				<tm-checkbox @change="autoDarkChange" :round='24' v-model='autoDark'></tm-checkbox>
			</template>
		</tm-cell>
		<!-- #ifdef H5 -->
		<tm-float-button @click="onChangeDark" :btn="{ icon: 'tmicon-ios-sunny', color: 'pink', linear: 'right' }"
			:offset="[50,150]">
		</tm-float-button>
		<!-- #endif -->
		<!-- #ifdef APP-VUE -->
		<tm-float-button @click="onChangeDark" :btn="{ icon: 'tmicon-ios-sunny', color: 'pink', linear: 'right' }"
			:offset="[50,50]">
		</tm-float-button>
		<!-- #endif -->
	</tm-app>
</template>

<script lang="ts" setup>
	import {
		ref,
		getCurrentInstance,
		computed,
		inject
	} from "vue"
	import {
		language
	} from "@/tmui/tool/lib/language"
	import { useTmpiniaStore } from '@/tmui/tool/lib/tmpinia';
	import tmApp from "@/tmui/components/tm-app/tm-app.vue"
	import tmSheet from "@/tmui/components/tm-sheet/tm-sheet.vue"
	import tmCell from "@/tmui/components/tm-cell/tm-cell.vue"
	import tmIcon from "@/tmui/components/tm-icon/tm-icon.vue"
	import tmCheckbox from "@/tmui/components/tm-checkbox/tm-checkbox.vue"
	import tmFloatButton from "@/tmui/components/tm-float-button/tm-float-button.vue"

	const store = useTmpiniaStore();
	const app = ref<InstanceType<typeof tmApp> | null>(null)
	const autoDark = ref(store.getAutoDark)
	function onChangeDark() {
		app.value?.setDark()
	}
	function autoDarkChange(e : boolean) {
		store.setTmAutoDark(e)
		// store.setTmVuetifyDark(e)
		autoDark.value = e;
	}
	function setLocal() {
		if (language("language") == "English-US") {
			uni.setLocale("zh-Hans")
		} else {
			uni.setLocale("en")
		}
	}
</script>

<style>

</style>