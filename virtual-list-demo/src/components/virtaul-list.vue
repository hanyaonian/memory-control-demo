<template>
  <div ref="root" style="height: 100%; overflow: hidden;">
    <span class="sticky-hint"> current list dom count: {{ count }} </span>
		<!-- virtual list container -->
		<div class="list-container" ref="listcontainer" @scroll="computeListRange">
			<!-- list phantom, fill the size -->
			<div class="list-phantom" :style="{ height: listHeight }"></div>
			<!-- list content, visble list content -->
			<ul class="list-content" ref="listcontent" :style="visibleArea">
				<li v-for="item in visbleList" :key="item">{{ item }}</li>
		    <div ref="bottom">bottom</div>
			</ul>
		</div>
  </div>
</template>

<script>
import { onMounted, onBeforeUnmount, ref, reactive, nextTick, computed } from 'vue';

// li 30px;
const LI_HEIGHT = 30;
const PAGE_SIZE = 30;

export default {
	setup() {
		const list = ref([]);
		const count = ref(0);


		/**
		 * list data 
		 */
		const bottom = ref(null);
		const listcontent = ref(null);
		let record = 0;

		const loadMoreData = () => {
			for (let i = record; i < record + PAGE_SIZE; i++) {
				list.value.push(i);
			}
			record += PAGE_SIZE;
			// get element count
			nextTick(() => {
				count.value = listcontent.value.getElementsByTagName('*').length;
				computeListRange();
			});
		};

		onMounted(() => {
			loadMoreData();
		});

		/**
		 * virtual list area
		 */
		const listcontainer = ref(null);

		const listHeight = computed(() => {
			const total = list.value.length;
			return `${(total) * LI_HEIGHT}px`;
		})

		const tail = ref(0);
		const start = ref(0);
		const visbleList = computed(() => {
			return list.value.slice(start.value, tail.value);
		});
		const visibleArea = reactive({
			webkitTransform: ''
		});
		const computeListRange = () => {
			const container = listcontainer.value;
			const { scrollTop, offsetHeight } = container;

			// visible area
			const visibleCount = Math.round(offsetHeight/LI_HEIGHT);
			start.value = Math.floor(scrollTop / LI_HEIGHT);
			// plus 2 防止速度太快出现缺失	
		  tail.value = visibleCount + start.value + 2;
			if (tail.value >= list.value.length) {
				loadMoreData();
			}
			visibleArea.webkitTransform = `translate3d(0, ${start.value * LI_HEIGHT}px, 0)`;
		};

		return {
			count,
			bottom,
			listcontainer,
			listcontent,
			visbleList,
			listHeight,
			computeListRange,
			visibleArea,
		}
	}
}
</script>

<style scoped>
.list-container {
	position: relative;
	height: 100%;
	overflow-y: scroll;
}
.list-content {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
}
</style>