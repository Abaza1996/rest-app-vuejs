<template>
    <div class="container">
        <Navbar />
        <form @click.prevent>
            <div class="row g-3 align-items-center">
                <div class="col-auto mx-auto d-block">
                    <h1>Delete All Categories</h1>
                    <hr />
                    <p class="text-danger">
                        Are you sure want to delete All Category for This
                        Location?
                    </p>
                    <br />
                    <div class="text-center">
                        <h1 class="mb0">{{ locName }}</h1>
                        <p class="text-muted">{{ locAddress }}</p>
                    </div>
                    <p class="text-danger">
                        When Deleting all categories, their related menu items
                        will be deleted as well...
                    </p>
                    <hr />
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto mx-auto d-block">
                    <button class="btn btn-info" @click="goBack()">
                        Go Back
                    </button>
                    &nbsp;&nbsp;&nbsp;
                    <button
                        class="btn btn-danger"
                        @click="deleteAllCategories()"
                    >
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
            locationId: this.$route.params.locationId,
            userId: "",
            locationData: [],
            successMessage: "",
            errorMessage: "",
            name: "",
            allItemsIdIs: [],
            allCatsIdIs: [],
            locName: "",
            locAddress: "",
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
            this.getLocationInfo(this.userId, this.locationId);
            let result = await axios.get(
                `http://localhost:3000/items?locId=${this.locationId}`
            );
            let resultLen = result.data.length;
            for (let i = 0; i < resultLen; i++) {
                this.allItemsIdIs.push(result.data[i].id);
            }
            let resultForCategories = await axios.get(
                `http://localhost:3000/categories?locationId=${this.locationId}`
            );
            let resultCatLen = resultForCategories.data.length;
            for (let i = 0; i < resultCatLen; i++) {
                this.allCatsIdIs.push(resultForCategories.data[i].id);
            }
        }
    },
    methods: {
        ...mapActions(["redirectTo"]),
        goBack() {
            this.$router.push({
                name: "ViewCategories",
                params: { locationId: this.locationId },
            });
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
        async deleteAllCategories() {
            let allCatsResults = [];
            for (let i = 0; i < this.allCatsIdIs.length; i++) {
                let result = await axios.delete(
                    `http://localhost:3000/categories/${this.allCatsIdIs[i]}`
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
                !allCatsResults.includes(false) &&
                !allItemsResults.includes(false)
            ) {
                this.successMessage =
                    "Category and related items are All Deleted..";
                this.errorMessage = "";
                setTimeout(() => {
                    this.$router.push({
                        name: "Home",
                    });
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
