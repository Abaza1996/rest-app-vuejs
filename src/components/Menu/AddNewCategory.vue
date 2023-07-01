<template>
    <Navbar />
    <div class="container">
        <router-link
            :to="{
                name: 'ViewCategories',
                params: { locationId: locationId },
            }"
        >
            <button class="btn btn-info">Back to Category</button> </router-link
        >&nbsp;&nbsp;
        <router-link :to="{ name: 'Menu', params: { locationId: locationId } }">
            <button class="btn btn-success">Back to Menu</button>
        </router-link>
        <br />
        <div class="text-center">
            <h1 class="mb0">{{ locName }}</h1>
            <p class="text-muted">{{ locAddress }}</p>
        </div>
        <br />
        <form @click.prevent>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <h1>Add New Category</h1>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <input
                        type="text"
                        class="form-control"
                        placeholder="Add Category Name"
                        v-model.trim="name"
                    />
                    <span class="error-feedback" v-if="v$.name.$error">{{
                        v$.name.$errors[0].$message
                    }}</span>
                </div>
            </div>
            <br />
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
                        @click="addCategory()"
                        class="btn btn-success"
                    >
                        Add Now
                    </button>
                </div>
            </div>
        </form>
    </div>
</template>

<script>
import Navbar from "@/components/Header/Navbar.vue";
import { mapActions, mapMutations } from "vuex";
import axios from "axios";
import useVuelidate from "@vuelidate/core";
import { required, minLength, maxLength } from "@vuelidate/validators";
export default {
    name: "AddNewCategory",
    components: {
        Navbar,
    },
    data() {
        return {
            v$: useVuelidate(),
            locationId: this.$route.params.locationId,
            userId: "",
            locName: "",
            locAddress: "",
            name: "",
            errorMessage: "",
            successMessage: "",
            listOfUserCategories: [],
        };
    },
    validations() {
        return {
            name: {
                required,
                minLength: minLength(3),
                maxLength: maxLength(15),
            },
        };
    },
    mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.userName = JSON.parse(user).name;
            this.userId = JSON.parse(user).id;
            this.canUserAccessThisLocation({
                userIdIs: this.userId,
                locationIdIs: this.locationId,
                redirectToPage: "Home",
            });
            this.getLocationInfo(this.userId, this.locationId);
            this.displayUserCategories(this.userId, this.locationId);
        }
    },
    methods: {
        ...mapMutations([
            "isLoggedInUser",
            "displayAllCategories",
            "canUserAccessThisLocation",
        ]),
        ...mapActions(["redirectTo"]),
        async displayUserCategories(userId, locId) {
            let result = await axios.get(
                `http://localhost:3000/categories?userId=${userId}&locationId=${locId}`
            );
            if (result.status == 200) {
                this.listOfUserCategories = result.data;
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
        async addCategory() {
            this.v$.$validate();
            let filterCategoryName = this.listOfUserCategories.filter(
                (v) =>
                    v.name.toLocaleLowerCase() == this.name.toLocaleLowerCase()
            );
            console.table(this.listOfUserCategories);
            if (!this.v$.$error) {
                if (filterCategoryName.length > 0) {
                    this.errorMessage =
                        "Category Name already exist, try anther name";
                    this.successMessage = "";
                } else {
                    let result = await axios.post(
                        `http://localhost:3000/categories`,
                        {
                            name: this.name,
                            userId: this.userId,
                            locationId: parseInt(this.locationId, 10),
                        }
                    );
                    if (result.status == 201) {
                        this.errorMessage = "";
                        this.successMessage = "Added New Category Successfully";
                        setTimeout(() => {
                            this.$router.push({
                                name: "ViewCategories",
                                params: { locationId: this.locationId },
                            });
                        }, 2000);
                    } else {
                        this.successMessage = "";
                        this.errorMessage =
                            "Something Went Wrong try again ...";
                    }
                }
            } else {
                this.successMessage = "";
                this.errorMessage = "You must fill in category name";
            }
        },
    },
};
</script>

<style></style>
