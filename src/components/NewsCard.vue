<template>
	<div class="ui card">
		<div class="content">
			<div class="right floated meta">{{ interval(new Date(date)) }}</div>
			<div class="left floated">{{ source }}</div>
		</div>
		<div class="image">
			<router-link :to="target">
				<img style="height:100%; width:100%" :src="image" />
			</router-link>
		</div>
		<div class="content">
			<router-link :to="target">
				<span class="header">{{ title }}</span>
			</router-link>
		</div>
	</div>
</template>

<style>
.ui.card {
	background-color: rgba(255, 255, 255, 0.8);
}
</style>

<script lang="ts">
import { Vue, Options } from "vue-class-component";
class Props {
	id!: number;
	title!: string;
	url!: string;
	image!: string;
	date!: string;
	source!: string;
}
@Options({
	props: {
		id: Number,
		title: String,
		url: String,
		image: String,
		date: String,
		source: String,
	},
})
export default class NewsCard extends Vue.with(Props) {
	get target() {
		return {
			name: "News",
			query: {
				id: this.id,
			},
		};
	}
	interval(from: Date, to?: Date): string {
		to = to ?? new Date();
		let delta = Math.round((to.getTime() - from.getTime()) / 3600000);
		let result = "";
		if (delta >= 8760) {
			result += `${Math.floor(delta / 8760)}年`;
			delta %= 8760;
		}
		if (delta >= 24) {
			result += `${Math.floor(delta / 24)}天`;
			delta %= 24;
		}
		result += `${delta}时`;
		return result;
	}
}
</script>
