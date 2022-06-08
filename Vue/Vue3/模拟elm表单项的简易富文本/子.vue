<template>
	<!-- 自定义输入框 -->
	<div class="input_wrapper">
		<span class="input_label" :class="isRequire && 'is_required'">{{
			label ? label + '：' : ''
		}}</span>
		<div
			contenteditable="plaintext-only"
			:data-text="placeholder"
			class="input_inner"
			@input="inputChange"
			@blur="inputBlur"
			@focus="inputFocus"
			@keydown="enter"
			ref="inner"
		></div>
		<div v-if="maxLength" class="max_length">{{ value.length }}/{{ maxLength }}</div>
		<div class="tips_message">{{ tips }}</div>
	</div>
</template>
<script lang="ts">
import { defineComponent, onMounted, ref, computed } from '@vue/composition-api';
export default defineComponent({
	props: {
		placeholder: {
			default: '请输入内容',
		},
		label: {
			default: '',
		},
		isRequire: {
			default: false,
		},
		rule: {
			default: () => [],
			type: Array,
		},
		value: {
			default: '',
			type: String,
		},
		maxLength: {
			default: undefined,
		},
		audit: {
			default: () => [],
			type: Array,
		},
		sensitive: {
			default: () => [],
			type: Array,
		},
	},
	setup(props, { emit }) {
		const tips = ref<string>(''); // tips
		const inner = ref(null); // ref
		onMounted(() => {
			props.value && (inner.value.innerText = props.value);
			for (let i = 0; i < props.rule.length; i++) {
				let _item = props.rule[i] as any;

				const requireFn = (trigger: string) => {
					inner.value.addEventListener(
						trigger,
						() => {
							if (!props.value.trim()) {
								tips.value = _item.message;
							} else {
								tips.value = '';
							}
						},
						true
					);
				};

				const minMaxFn = (trigger: string) => {
					inner.value.addEventListener(
						trigger,
						() => {
							let vl = props.value.length;
							if (props.value.trim()) {
								if (vl < _item.min || vl > _item.max) {
									tips.value = _item.message;
								} else {
									tips.value = '';
								}
							}
						},
						true
					);
				};
				if (Object.prototype.toString.call(_item.trigger) === '[object String]') {
					if (_item.required) {
						requireFn(_item.trigger);
					}

					if (_item.min || _item.max) {
						minMaxFn(_item.trigger);
					}
				}
				if (Object.prototype.toString.call(_item.trigger) === '[object Array]') {
					_item.trigger.map((item) => {
						if (_item.required) {
							requireFn(item);
						}

						if (_item.min || _item.max) {
							minMaxFn(item);
						}
					});
				}
				// 如果是自定义规则，就走这边
				if (_item.validity) {
				}
			}
		});

		const inputChange = () => {
			if (props.maxLength && inner.value.innerText.length >= props.maxLength) {
				inner.value.innerText = inner.value.innerText.slice(0, props.maxLength);
				if (window.getSelection) {
					let range = window.getSelection();
					range.selectAllChildren(inner.value); //range 选择obj下所有子内容
					range.collapseToEnd(); //光标移至最后
				}
			}
			emit('input', inner.value.innerText);
			inner.value.className = 'input_inner is_focus';
		};
		const inputFocus = (e) => {
			const tar = e.target || e.srcEvent;
			if (!tar.className.includes('is_focus')) {
				tar.className += ' is_focus';
			}
		};
		const inputBlur = (e) => {
			const tar = e.target || e.srcEvent;

			if (tar.className.includes('is_focus')) {
				tar.className = 'input_inner';
			}
			let text = tar.innerText;
			props.sensitive.map((item) => {
				text = text.replace(new RegExp(item, 'g'), `<span class='sensitive_text'>${item}</span>`);
			});
			props.audit.map((item) => {
				text = text.replace(new RegExp(item, 'g'), `<span class='audit_text'>${item}</span>`);
			});
			if ((text.includes('audit_text') && !tar.className.includes('is_audit')) || tips.value) {
				tar.className += ' is_audit';
			} else if (text.includes('sensitive_text') && !tar.className.includes('is_sensitive')) {
				tar.className += ' is_sensitive';
			} else {
				tar.className = 'input_inner';
			}
			tar.innerHTML = text;
			emit('input', tar.innerText);
		};

		const enter = (e) => {
			if (e.keyCode == 13) {
				e.preventDefault();
				tar.blur();
			}
		};

		return {
			inputFocus,
			inputBlur,
			inputChange,
			enter,
			tips,
			inner,
		};
	},
});
</script>

<style lang="scss">
.input_wrapper {
	margin-bottom: 40px;
	height: 40px;
	width: 600px;
	position: relative;

	.input_label {
		width: 90px;
		margin-right: 12px;
		box-sizing: border-box;
		font-size: 14px;
		font-weight: 500;
		color: rgba(0, 0, 0, 0.85);
		float: left;
		&:before {
			content: '*';
			color: transparent;
			margin-right: 4px;
		}
	}
	.input_inner {
		-webkit-user-select: text;
		float: left;
		background-color: #fff;
		background-image: none;
		border-radius: 4px;
		border: 1px solid #dcdfe6;
		color: #606266;
		font-size: inherit;
		box-sizing: border-box;
		padding-left: 15px;
		padding-right: 50px;
		height: 40px;
		line-height: 40px;
		transition: border-color 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
		border-radius: 6px;
		width: calc(100% - 102px);
		outline: none;
		&:empty:before {
			content: attr(data-text);
			color: rgba(0, 0, 0, 0.08);
		}
	}

	.is_required {
		&:before {
			content: '*';
			color: #f56c6c;
			margin-right: 4px;
		}
	}

	.is_focus {
		border: 1px solid #1860ff;
	}

	.is_audit {
		border: 1px solid red;
	}

	.is_sensitive {
		border: 1px solid yellow;
	}
}
.sensitive_text {
	color: yellow;
}
.audit_text {
	color: red;
}

.max_length {
	position: absolute;
	right: 15px;
	height: 40px;
	line-height: 40px;
	color: rgba(0, 0, 0, 0.2);
	font-weight: 400;
	font-size: 14px;
}

.tips_message {
	position: absolute;
	bottom: -25px;
	margin-left: 90px;
	padding-left: 12px;
	color: red;
}
</style>
