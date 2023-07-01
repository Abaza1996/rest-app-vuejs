<template>
    <div class="container">
        <Navbar />
        <form @click.prevent>
            <div class="row g-3 align-items-center">
                <div class="col-auto mx-auto d-block">
                    <h1>Delete All locations</h1>
                    <hr />
                    <p class="text-danger">
                        Are you sure want to delete all locations?
                    </p>
                    <p class="text-danger">
                        It will also delete all related items and
                        categories...Be careful!
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
                        @click="deleteAllLocations()"
                    >
                        Delete All
                    </button>
                </div>
            </div>
            <br />
            <div class="row g-3 align-items-center">
                <div
                    class="col-auto d-block mx-auto alert alert-warning"
                    v-if="successMessage.length > 0"
                >
                    {{ successMessage }}
                </div>
                <div
                    class="col-auto d-block mx-auto alert alert-success"
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
import axios from "axios";
import { mapActions } from "vuex";
export default {
    name: "DeleteAllLocations",
    components: {
        Navbar,
    },
    data() {
        return {
            userId: "",
            successMessage: "",
            errorMessage: "",
            allLocationsId: [],
            allItemsIdIs: [],
            allCatsIdIs: [],
        };
    },
    async mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.userId = JSON.parse(user).id;
            let resultLoc = await axios.get(
                `http://localhost:3000/locations?userId=${this.userId}`
            );
            let resultLocLen = resultLoc.data.length;
            for (let i = 0; i < resultLocLen; i++) {
                this.allLocationsId.push(resultLoc.data[i].id);
            }
            let resultCatLoc = await axios.get(
                `http://localhost:3000/categories?userId=${this.userId}`
            );
            let resultCatLocLen = resultCatLoc.data.length;
            for (let i = 0; i < resultCatLocLen; i++) {
                this.allCatsIdIs.push(resultCatLoc.data[i].id);
            }
            let resultItemsLoc = await axios.get(
                `http://localhost:3000/items?userId=${this.userId}`
            );
            let resultItemsLocLen = resultItemsLoc.data.length;
            for (let i = 0; i < resultItemsLocLen; i++) {
                this.allItemsIdIs.push(resultItemsLoc.data[i].id);
            }
        }
    },
    methods: {
        ...mapActions(["redirectTo"]),
        goBack() {
            this.redirectTo({ val: "Home" });
        },
        async deleteAllLocations() {
            let allItemsResults = [];
            for (let i = 0; i < this.allItemsIdIs.length; i++) {
                let result1 = await axios.delete(
                    `http://localhost:3000/items/${this.allItemsIdIs[i]}`
                );
                if (result1.data == 200) {
                    allItemsResults.push(true);
                } else {
                    allItemsResults.push(false);
                }
            }
            let allCatsResults = [];
            for (let i = 0; i < this.allCatsIdIs.length; i++) {
                let result2 = await axios.delete(
                    `http://localhost:3000/categories/${this.allCatsIdIs[i]}`
                );
                if (result2.data == 200) {
                    allCatsResults.push(true);
                } else {
                    allCatsResults.push(false);
                }
            }
            let allResults = [];
            for (let i = 0; i < this.allLocationsId.length; i++) {
                let result = await axios.delete(
                    `http://localhost:3000/locations/${this.allLocationsId[i]}`
                );
                if (result.data == 200) {
                    allResults.push(true);
                } else {
                    allResults.push(false);
                }
            }
            if (
                !allResults.includes(false) &&
                !allCatsResults.includes(false) &&
                !allItemsResults.includes(false)
            ) {
                this.successMessage = "All Location are deleted..";
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
