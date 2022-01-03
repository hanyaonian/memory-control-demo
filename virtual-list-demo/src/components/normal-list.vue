<template>
	<div class="scroll-list">
		<span class="sticky-hint"> current list dom count: {{ count }} </span>
		<ul ref="container">
			<li v-for="(item, i) in list" :key="i"> {{ i }} </li>
		</ul>
		<div ref="bottom">bottom</div>
	</div>
</template>

<script>
import { onMounted, onBeforeUnmount, ref, nextTick } from 'vue';

export default {
	setup() {
		const list = ref([]);
		const count = ref(0);
		const container = ref(null);

		const loadMoreData = (e) => {
			const [ el ] = e;
			if (!(el.intersectionRatio > 0)) {
				return ;
			}
			for (let i = 0; i < 30; i++) {
				list.value.push(i);
			}
			// get element count
			nextTick(() => {
				count.value = container.value.getElementsByTagName('*').length;
			})
		};

		const observer = new IntersectionObserver(loadMoreData, {
			root: document.querySelector('#scrollArea'),
			rootMargin: '30px',
			threshold: 0.9
		});

		const bottom = ref(null);
		onMounted(() => {
			observer.observe(bottom.value);
		});

		onBeforeUnmount(() => {
			observer.disconnect();
		});

		return {
			count,
			container,
			bottom,
			list,
		}
	}
}
</script>
