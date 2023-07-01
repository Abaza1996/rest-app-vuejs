<template>
    <div class="container">
        <Navbar />
        <form @click.prevent>
            <div class="row g-3 align-items-center">
                <div class="col-auto mx-auto d-block">
                    <h1>Delete Restaurant {{ deleteLocation }}</h1>
                    <hr />
                    <p class="text-danger">
                        Are you sure want to delete this location?
                    </p>
                    {{ name }}<br />
                    {{ phone }}<br />
                    {{ address }}<br />
                    <hr />
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto mx-auto d-block">
                    <button class="btn btn-info" @click="goBack()">
                        Go Back
                    </button>
                    &nbsp;&nbsp;&nbsp;
                    <button class="btn btn-danger" @click="deleteRestaurant()">
                        Delete now
                    </button>
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
                    class="col-auto d-block mx-auto alert alert-warning"
                    v-if="errorMessage.length > 0"
                >
                    {{ errorMessage }}
                </div>
            </div>
        </form>
    </div>
</template>
<script>
import Navbar from "@/components/Header/Navbar.vue";
import { mapActions } from "vuex";
import axios from "axios";
export default {
    name: "DeleteLocation",
    components: {
        Navbar,
    },
    data() {
        return {
            name: "",
            phone: "",
            address: "",
            userId: "",
            deleteLocation: "",
            locationData: [],
            successMessage: "",
            errorMessage: "",
            allItemsIdIs: [],
            allCatsIdIs: [],
        };
    },
    async mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.deleteLocation = this.$route.params.locationId;
            this.userId = JSON.parse(user).id;
            this.canCurrentUserAccessThisLocation();
            let result = await axios.get(
                `http://localhost:3000/items?locId=${this.locationId}`
            );
            let resultlLen = result.data.length;
            for (let i = 0; i < resultlLen; i++) {
                this.allItemsIdIs.push(result.data[i].id);
            }
            let resultForCategories = await axios.get(
                `http://localhost:3000/categories?locationId=${this.locationId}`
            );
            let resultCatLen = result.data.length;
            for (let i = 0; i < resultCatLen; i++) {
                this.allCatsIdIs.push(resultForCategories.data[i].id);
            }
        }
    },
    methods: {
        ...mapActions(["redirectTo"]),
        goBack() {
            this.redirectTo({ val: "Home" });
        },
        async canCurrentUserAccessThisLocation() {
            let result = await axios.get(
                `http://localhost:3000/locations?id=${this.deleteLocation}&userId=${this.userId}`
            );
            if (result.status == 200 && result.data.length > 0) {
                this.locationData = result.data;
                this.name = this.locationData[0].name;
                this.phone = this.locationData[0].phone;
                this.address = this.locationData[0].address;
            } else {
                this.redirectTo({ val: "Home" });
            }
        },
        async deleteRestaurant() {
            let result = await axios.delete(
                `http://localhost:3000/locations/${this.deleteLocation}`
            );
            let allCatsResults = [];
            for (let i = 0; i < this.allCatsIdIs.length; i++) {
                let result = await axios.delete(
                    `http://localhost:3000/categories/${this.allItemsIdIs[i]}`
                );
                if (result.status == 200) {
                    allCatsResults.push(true);
                } else {
                    allCatsResults.push(false);
                }
            }
            let allItemsResults = [];
            for (let i = 0; i < this.allItemsIdIs.length; i++) {
                let result = await axios.delete(
                    `http://localhost:3000/items/${this.allItemsIdIs[i]}`
                );
                if (result.status == 200) {
                    allItemsResults.push(true);
                } else {
                    allItemsResults.push(false);
                }
            }
            if (
                result.status == 200 &&
                !allCatsResults.includes(false) &&
                !allItemsResults.includes(false)
            ) {
                this.successMessage = "Location is deleted..";
                this.errorMessage = "";
                setTimeout(() => {
                    this.redirectTo({ val: "Home" });
                }, 2000);
            } else {
                this.errorMessage = "";
                this.successMessage = "Something Went Wrong, try again";
            }
        },
    },
};
</script>
<style></style>
