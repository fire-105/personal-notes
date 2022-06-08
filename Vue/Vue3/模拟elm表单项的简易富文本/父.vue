<template>
	<div>
		<CustomziInput
			label="necessary"
			v-model="someValue"
			:isRequire="true"
			:maxLength="25"
			placeholder="请输入内容"
			key="1"
			:audit="['violence', 'fantasticy']"
			:sensitive="['pornography', 'forg']"
			:rule="inputRule"
		/>
		<CustomziInput label="内容审核" v-model="someValue1" placeholder="请输入内容" />
	</div>
</template>

<script lang="ts">
import { defineComponent, onBeforeMount, ref } from '@vue/composition-api';

export default defineComponent({
	components: {
		CustomziInput,
	},
	setup() {
		const someValue = ref<string>('enen');
		const someValue1 = ref<string>('11');
		const inputRule = [
			{ required: true, message: 'Please enter required fields', trigger: 'blur' },
			{ min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' },
		];
		return {
			someValue,
			someValue1,
			inputRule,
		};
	},
});
</script>