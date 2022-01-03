<template>
	<li> {{ d.name }} </li>
</template>

<script>
import { onMounted, onBeforeUnmount, toRefs } from 'vue';

export default {
	props: {
		d: {
			type: Object,
		},
		leaked: {
			type: Boolean,
		}
	},
	setup(props) {
		const { leaked, d } = toRefs(props);

		onMounted(() => {
			document.addEventListener('click', showClickHint);
		});

		onBeforeUnmount(() => {
			if (!leaked.value) {
				document.removeEventListener('click', showClickHint);
			}
		});

		const showClickHint = (e) => {
			console.log('document is clicked');
		};

		return {
			d,
		};
	}
}
</script>

<style>

</style>