<template>
	<div class="container my-5">
		<b-card-group deck>
			<b-card
				title="Description"
				:img-src="`/${currentItem.img}`"
				img-top
				style="border: none"
			>
				<b-card-text>{{ currentItem.description }}</b-card-text>
			</b-card>

			<b-card style="border: none">
				<b-card-title class="display-4">{{
					currentItem.item
				}}</b-card-title>
				<b-card-sub-title
					><span class="h2"
						>Price: ₹ {{ currentItem.price.toFixed(2) }}</span
					></b-card-sub-title
				>
				<b-card-text class="mt-2">
					<b-form inline @submit.prevent="">
						<b-form-input
							type="number"
							min="1"
							v-model="count"
							style="width: 4rem"
						>
						</b-form-input>
						<b-button
							class="ml-3"
							style="width: 75%"
							variant="success"
							@click="addToCart"
						>
							Add to Cart - ₹ {{ combinedPrice }}
						</b-button>
					</b-form>
					<b-form-group label="Options" class="my-3">
						<b-form-radio
							v-for="option in currentItem.options"
							:key="option"
							:id="option"
							:value="option"
							v-model="$v.itemOptions.$model"
							>{{ option }}
						</b-form-radio>
					</b-form-group>

					<b-form-group label="Addons">
						<b-form-checkbox
							v-for="addon in currentItem.addOns"
							:key="addon"
							:id="addon"
							:value="addon"
							v-model="$v.itemAddons.$model"
						>
							{{ addon }}
						</b-form-checkbox>
					</b-form-group>
				</b-card-text>
			</b-card>
		</b-card-group>
	</div>
</template>

<script>
import { mapState } from "vuex";
import { required } from "vuelidate/lib/validators";
export default {
	data() {
		return {
			id: this.$route.params.id,
			count: 1,
			itemOptions: "",
			itemAddons: [],
			itemSizeAndCost: [],
			toastCount: 0,
			cartSubmitted: false,
		};
	},
	validations: {
		itemOptions: {
			required,
		},
		itemAddons: {
			required,
		},
	},
	computed: {
		...mapState(["foodData"]),
		currentItem() {
			let result;
			for (let i = 0; i < this.foodData.length; i++) {
				for (let j = 0; j < this.foodData[i].menu.length; j++) {
					if (this.foodData[i].menu[j].id === this.id) {
						result = this.foodData[i].menu[j];
						break;
					}
				}
			}
			return result;
		},
		combinedPrice() {
			let total = this.count * this.currentItem.price;
			return total.toFixed(2);
		},
	},
	methods: {
		addToCart() {
			let cartOutput = {
				item: this.currentItem.item,
				count: this.count,
				options: this.itemOptions,
				addons: this.itemAddons,
				combinedPrice: this.combinedPrice,
			};

			let addOnError = this.$v.itemAddons.$invalid;
			let optionsError = this.currentItem.options
				? this.$v.itemOptions.$invalid
				: false;
			let toastObj = {};
			if (addOnError || optionsError) {
				this.errors = true;
				toastObj["title"] = "Order Failed";
				toastObj["variant"] = "danger";
				toastObj["message"] =
					"Please add options and addons before continuing";
			} else {
				toastObj["title"] = "Order Submitted";
				toastObj["variant"] = "success";
				toastObj["message"] = "Check out more !!!";
				this.cartSubmitted = true;
				this.$store.commit("addToCart", cartOutput);
			}
			this.makeToast(toastObj);
		},
		makeToast(toastObj) {
			this.$bvToast.toast(toastObj["message"], {
				to: "/resturants",
				title: toastObj["title"],
				autoHideDelay: 5000,
				appendToast: true,
				variant: toastObj["variant"],
				solid: true,
			});
		},
	},
};
</script>

<style lang="scss" scoped></style>
