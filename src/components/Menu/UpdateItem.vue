<template>
    <Navbar />
    <div class="container">
        <div class="clearfix"></div>
        <router-link
            :to="{
                name: 'Menu',
                params: { locationId: locationId },
            }"
        >
            <button type="button" class="float-start btn btn-info">
                Back To Menu
            </button>
        </router-link>
        <div class="clearfix"></div>
        <br />
        <div class="text-center">
            <h1 class="mb0">{{ locName }}</h1>
            <p class="text-muted">{{ locAddress }}</p>
        </div>
        <form @click.prevent>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <hr />
                    <h1>Add New Item</h1>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <div
                        class="w250 form-floating mb-3"
                        :class="{
                            'form-group-error': v$.itemName.$error,
                        }"
                    >
                        <input
                            type="text"
                            class="form-control"
                            id="floatItemName"
                            placeholder="Enter Item Name"
                            v-model.trim="itemName"
                        />
                        <label for="floatItemName">Enter Item Name</label>
                        <span
                            class="error-feedback"
                            v-if="v$.itemName.$error"
                            >{{ v$.itemName.$errors[0].$message }}</span
                        >
                    </div>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <div
                        class="w250 form-floating mb-3"
                        :class="{
                            'form-group-error': v$.itemPrice.$error,
                        }"
                    >
                        <input
                            type="Number"
                            class="form-control"
                            id="floatItemPrice"
                            placeholder="Enter Item Price"
                            v-model.trim="itemPrice"
                        />
                        <label for="floatItemPrice">Enter Item Price</label>
                        <span
                            class="error-feedback"
                            v-if="v$.itemPrice.$error"
                            >{{ v$.itemPrice.$errors[0].$message }}</span
                        >
                    </div>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <div
                        class="w250 form-floating mb-3"
                        :class="{
                            'form-group-error': v$.itemsQty.$error,
                        }"
                    >
                        <input
                            type="Number"
                            class="form-control"
                            id="floatItemQty"
                            placeholder="Enter Item Quantities"
                            v-model.trim="itemsQty"
                        />
                        <label for="floatItemQty">Enter Item Quantities</label>
                        <span
                            class="error-feedback"
                            v-if="v$.itemsQty.$error"
                            >{{ v$.itemsQty.$errors[0].$message }}</span
                        >
                    </div>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <div
                        class="form-floating mb-3"
                        :class="{
                            'form-group-error': v$.description.$error,
                        }"
                    >
                        <textarea
                            class="w250 h200 form-control"
                            placeholder="Enter Item Descriptions"
                            id="floatItemDescriptions"
                            v-model.trim="description"
                        ></textarea>
                        <label for="floatItemDescriptions"
                            >Enter Item Descriptions</label
                        >
                        <span
                            class="error-feedback"
                            v-if="v$.description.$error"
                            >{{ v$.description.$errors[0].$message }}</span
                        >
                    </div>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <div
                        class="form-floating mb-3"
                        :class="{
                            'form-group-error': v$.pickedCategory.$error,
                        }"
                    >
                        <select
                            class="w250 form-select"
                            id="floatItemCategory"
                            v-model.trim="pickedCategory"
                        >
                            <option
                                v-for="(cat, i) in listOfCategories"
                                :key="i"
                                :value="cat.id"
                            >
                                {{ cat.name }}
                            </option>
                        </select>
                        <label for="floatItemCategory"
                            >Select Category Name</label
                        >
                    </div>
                    <span
                        class="error-feedback"
                        v-if="v$.pickedCategory.$error"
                        >{{ v$.pickedCategory.$errors[0].$message }}</span
                    >
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div
                    class="col-auto d-block mx-auto alert alert-success"
                    v-if="successMessage.length > 0"
                >
                    {{ successMessage }}
                </div>
                <div
                    class="col-auto d-block mx-auto alert alert-danger"
                    v-if="errorMessage.length > 0"
                >
                    {{ errorMessage }}
                </div>
            </div>
            <br />
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <button
                        type="button"
                        @click="UpdateItem()"
                        class="w250 btn btn-warning"
                    >
                        Update Now
                    </button>
                </div>
            </div>
        </form>
    </div>
</template>

<script>
import Navbar from "@/components/Header/Navbar.vue";
import { mapActions, mapState, mapMutations } from "vuex";
import useVuelidate from "@vuelidate/core";
import { between, maxLength, minLength, required } from "@vuelidate/validators";
import axios from "axios";
export default {
    components: { Navbar },
    // eslint-disable-next-line vue/multi-word-component-names
    name: "UpdateItem",
    data() {
        return {
            v$: useVuelidate(),
            userId: "",
            userName: "",
            locationId: this.$route.params.locationId,
            itemId: this.$route.params.itemId,
            locName: "",
            locAddress: "",
            successMessage: "",
            errorMessage: "",
            pickedCategory: "",
            itemName: "",
            itemsQty: 1,
            description: "",
            itemPrice: "",
        };
    },
    computed: {
        ...mapState([
            "isUserLoggedIn",
            "loggedInUserId",
            "numOfCategorise",
            "listOfCategories",
        ]),
    },
    validations() {
        return {
            itemName: { required, minLength: minLength(6) },
            description: {
                required,
                minLength: minLength(6),
                maxLength: maxLength(120),
            },
            itemsQty: { required, between: between(1, 100000) },
            itemPrice: { required, between: between(0, 1000000) },
            pickedCategory: { required },
        };
    },
    async mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.userName = JSON.parse(user).name;
            this.userId = JSON.parse(user).id;
            this.isLoggedInUser();
            // this.locationId = this.$route.params.locationID;
            this.displayAllCategories({
                userIdIs: this.userId,
                locationIdIs: this.locationId,
            });
            this.canUserAccessThisLocation({
                userIdIs: this.userId,
                locationIdIs: this.locationId,
                redirectToPage: "Home",
            });
            this.canUserAccessThisItem({
                userIdIs: this.userId,
                locationIdIs: this.locationId,
                itemIdIs: this.itemId,
                redirectToPage: "Home",
            });
            this.getLocationInfo(this.userId, this.locationId);
            this.getItemInfo(this.userId, this.locationId, this.itemId);
        }
    },
    methods: {
        ...mapMutations([
            "isLoggedInUser",
            "displayAllCategories",
            "canUserAccessThisLocation",
            "canUserAccessThisItem",
        ]),
        ...mapActions(["redirectTo"]),
        async getItemInfo(userId, locId, itemId) {
            let result = await axios.get(
                `http://localhost:3000/items?userId=${userId}&locationId=${locId}&id=${itemId}`
            );
            let resultData = result.data[0];
            if (result.status == 200) {
                this.description = resultData.description;
                this.itemName = resultData.name;
                this.itemsQty = resultData.qty;
                this.itemPrice = resultData.price;
                this.pickedCategory = resultData.catId;
            }
        },
        async getLocationInfo(userId, locId) {
            let result = await axios.get(
                `http://localhost:3000/locations?userId=${userId}&id=${locId}`
            );
            let locDetails = [];
            if (result.status == 200 && result.data.length > 0) {
                locDetails = result.data;
                this.locName = locDetails[0].name;
                this.locAddress = locDetails[0].address;
            }
        },
        async UpdateItem() {
            this.v$.$validate();
            if (!this.v$.$error) {
                console.log("Validated");
                let results = await axios.put(
                    `http://localhost:3000/items/${this.itemId}`,
                    {
                        name: this.itemName,
                        price: parseFloat(this.itemPrice).toFixed(2),
                        description: this.description,
                        qty: parseInt(this.itemsQty, 10),
                        userId: this.userId,
                        locId: parseInt(this.locationId, 10),
                        catId: this.pickedCategory,
                    }
                );
                if (results.status == 200) {
                    this.errorMessage = "";
                    this.successMessage = "Item Updated Successfully ...";
                    setTimeout(() => {
                        this.$router.push({
                            name: "Menu",
                            params: { locationId: this.locationId },
                        });
                    }, 2000);
                } else {
                    this.successMessage = "";
                    this.errorMessage =
                        "Something Went Wrong , please try again!";
                    console.log("Failed Updated this Item");
                }
            } else {
                this.successMessage = "";
                this.errorMessage = "You must fill in All required fields!";
            }
        },
    },
};
</script>
<style>
.mb0 {
    margin-bottom: 0;
}
.w250 {
    min-width: 250px;
}
.h200 {
    min-height: 200px !important;
}
.form-group-error {
    color: red;
}
.form-group-error input,
.form-group-error select,
.form-group-error textarea {
    border-color: red;
}
</style>
