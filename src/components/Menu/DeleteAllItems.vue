<template>
    <div class="container">
        <Navbar />
        <form @click.prevent>
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
            <div class="row g-3 align-items-center">
                <div class="col-auto mx-auto d-block">
                    <div class="clearfix"></div>
                    <div class="text-center">
                        <h1 class="mb0">{{ locName }}</h1>
                        <p class="text-muted">{{ locAddress }}</p>
                    </div>
                    <div class="clearfix"></div>
                    <p class="text-danger">
                        Are you sure want to delete all Items for above
                        location?
                    </p>
                    <p class="text-danger">
                        When Deleting this items, you will no longer see them...
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
                    <button class="btn btn-danger" @click="deleteAllItems()">
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
import { mapActions, mapMutations } from "vuex";
import axios from "axios";
export default {
    name: "DeleteAllItems",
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
            locName: "",
            locAddress: "",
            allItemsIdIs: [],
            myResult: "",
        };
    },
    async mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.userId = JSON.parse(user).id;
            this.canCurrentUserAccessThisLocation();
            let result = await axios.get(
                `http://localhost:3000/items?userId=${this.userId}&locId=${this.locationId}`
            );
            this.myResult = result.data;
            let resultLen = this.myResult.length;
            for (let i = 0; i < resultLen; i++) {
                this.allItemsIdIs.push(result.data[i].id);
            }
            console.table(this.allItemsIdIs);
        }
    },
    methods: {
        ...mapMutations(["canUserAccessThisItem"]),
        ...mapActions(["redirectTo"]),
        goBack() {
            this.$router.push({
                name: "Menu",
                params: { locationId: this.locationId },
            });
        },
        async canCurrentUserAccessThisLocation() {
            let result = await axios.get(
                `http://localhost:3000/locations?id=${this.locationId}&userId=${this.userId}`
            );
            if (result.status == 200 && result.data.length > 0) {
                this.locationData = result.data;
                this.locName = this.locationData[0].name;
                this.locAddress = this.locationData[0].address;
            } else {
                this.redirectTo({ val: "Home" });
            }
        },
        async deleteAllItems() {
            let allResults = [];
            for (let i = 0; i < this.allItemsIdIs.length; i++) {
                let result = await axios.delete(
                    `http://localhost:3000/items/${this.allItemsIdIs[i]}`
                );
                if (result.status == 200) {
                    allResults.push(true);
                } else {
                    allResults.push(false);
                }
            }
            if (!allResults.includes(false)) {
                this.successMessage = "All Items are deleted..";
                this.errorMessage = "";
                setTimeout(() => {
                    this.$router.push({
                        name: "Menu",
                        params: { locationId: this.locationId },
                    });
                }, 2000);
            } else {
                this.successMessage = "";
                this.errorMessage = "Something Went Wrong, try again";
            }
        },
    },
};
</script>
<style></style>
