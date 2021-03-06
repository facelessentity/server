<template>
	<li :id="item.id" :class="[{'icon-loading-small': item.loading, 'open': item.opened, 'collapsible': item.collapsible&&item.children&&item.children.length>0 }, item.classes]">

		<!-- Bullet -->
		<div v-if="item.bullet" class="app-navigation-entry-bullet" :style="{ backgroundColor: item.bullet }"></div>

		<!-- Main link -->
		<a v-if="item.href" :href="(item.href) ? item.href : '#' " @click="toggleCollapse" :class="item.icon" >
			<img v-if="item.iconUrl" :alt="item.text" :src="item.iconUrl">
			{{item.text}}
		</a>

		<!-- Router link if specified. href OR router -->
		<router-link :to="item.router" v-else-if="item.router" :class="item.icon" >
			<img v-if="item.iconUrl" :alt="item.text" :src="item.iconUrl">
			{{item.text}}
		</router-link>	

		<!-- Popover, counter and button(s) -->
		<div v-if="item.utils" class="app-navigation-entry-utils">
			<ul>
				<!-- counter -->
				<li v-if="Number.isInteger(item.utils.counter)"
					class="app-navigation-entry-utils-counter">{{item.utils.counter}}</li>

				<!-- first action if only one action and counter -->
				<li v-if="item.utils.actions && item.utils.actions.length === 1 && Number.isInteger(item.utils.counter)"
					class="app-navigation-entry-utils-menu-button">
					<button @click="item.utils.actions[0].action" :class="item.utils.actions[0].icon" :title="item.utils.actions[0].text"></button>
				</li>

				<!-- second action only two actions and no counter -->
				<li v-else-if="item.utils.actions && item.utils.actions.length === 2 && !Number.isInteger(item.utils.counter)"
					v-for="action in item.utils.actions" :key="action.action"
					class="app-navigation-entry-utils-menu-button">
					<button @click="action.action" :class="action.icon" :title="action.text"></button>
				</li>

				<!-- menu if only at least one action and counter OR two actions and no counter-->
				<li v-else-if="item.utils.actions && item.utils.actions.length > 1 && (Number.isInteger(item.utils.counter) || item.utils.actions.length > 2)"
					class="app-navigation-entry-utils-menu-button">
					<button v-click-outside="hideMenu" @click="showMenu" ></button>
				</li>
			</ul>
		</div>

		<!-- if more than 2 actions or more than 1 actions with counter -->
		<div v-if="item.utils && item.utils.actions && item.utils.actions.length > 1 && (Number.isInteger(item.utils.counter) || item.utils.actions.length > 2)"
			 class="app-navigation-entry-menu" :class="{ 'open': openedMenu }">
			 <popover-menu :menu="item.utils.actions"/>
		</div>

		<!-- undo entry -->
		<div class="app-navigation-entry-deleted" v-if="item.undo">
			<div class="app-navigation-entry-deleted-description">{{item.undo.text}}</div>
			<button class="app-navigation-entry-deleted-button icon-history" :title="t('settings', 'Undo')"></button>
		</div>

		<!-- edit entry -->
		<div class="app-navigation-entry-edit" v-if="item.edit">
			<form>
				<input type="text" v-model="item.text">
				<input type="submit" value="" class="icon-confirm">
				<input type="submit" value="" class="icon-close" @click.stop.prevent="cancelEdit">
			</form>
		</div>

		<!-- if the item has children, inject the component with proper data -->
		<ul v-if="item.children">
			<navigation-item v-for="(item, key) in item.children" :item="item" :key="key" />
		</ul>
	</li>
</template>

<script>
import popoverMenu from '../popoverMenu';
import ClickOutside from 'vue-click-outside';
import Vue from 'vue';

export default {
	name: 'navigationItem',
	props: ['item'],
	components: {
		popoverMenu
	},
	directives: {
		ClickOutside
	},
	data() {
		return {
			openedMenu: false
		}
	},
	methods: {
		showMenu() {
			this.openedMenu = true;
		},
		hideMenu() {
			this.openedMenu = false;
		},
		toggleCollapse() {
			// if item.opened isn't set, Vue won't trigger view updates https://vuejs.org/v2/api/#Vue-set
			// ternary is here to detect the undefined state of item.opened
			Vue.set(this.item, 'opened', this.item.opened ? !this.item.opened : true);
		},
		cancelEdit() {
			// remove the editing class
			if (Array.isArray(this.item.classes))
				this.item.classes = this.item.classes.filter(item => item !== 'editing');
		}
	},
	mounted() {
		// prevent click outside event with popupItem.
		this.popupItem = this.$el;
	},
}
</script>
