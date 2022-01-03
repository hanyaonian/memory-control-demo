<template>
	<ul>
  	<label for="leakMode"> 泄漏模式 </label>
  	<input name="leakMode" type="checkbox" v-model="leakMode" />
		<list-item 
			v-for="item in listData"
			:leaked="leakMode"
			:key="item.key"
			:d="item"
		/>
	</ul>
</template>

<script>
import { ref, toRefs } from 'vue';
import ListItem from './list-item.vue';

export default {
	components: { ListItem },
	props: {
		listData: {
			type: Array,
			default: []
		}
	},
	setup(props) {
    const leakMode = ref(false);
		const { listData } = toRefs(props);

		return {
			listData,
			leakMode,
		}
	},
}
</script>

<style>

</style>