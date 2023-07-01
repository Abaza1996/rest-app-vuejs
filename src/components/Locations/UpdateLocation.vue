<template>
    <Navbar />
    <form @click.prevent>
        <div class="row g-3 align-items-center">
            <div class="col-auto d-block mx-auto">
                <h1>Update Restaurant</h1>
            </div>
        </div>
        <div class="row g-3 align-items-center">
            <div class="col-auto d-block mx-auto">
                <input
                    type="text"
                    class="form-control"
                    placeholder="Restaurant Name"
                    v-model.trim="state.name"
                />
                <span class="error-feedback" v-if="v$.name.$error">{{
                    v$.name.$errors[0].$message
                }}</span>
            </div>
        </div>
        <br />
        <div class="row g-3 align-items-center">
            <div class="col-auto d-block mx-auto">
                <input
                    type="text"
                    class="form-control"
                    placeholder="Phone Number"
                    v-model.trim="state.phone"
                />
                <span class="error-feedback" v-if="v$.phone.$error">{{
                    v$.phone.$errors[0].$message
                }}</span>
            </div>
        </div>
        <br />
        <div class="row g-3 align-items-center">
            <div class="col-auto d-block mx-auto">
                <input
                    type="text"
                    class="form-control"
                    placeholder="Restaurant Address"
                    v-model="state.address"
                />
                <span class="error-feedback" v-if="v$.address.$error">{{
                    v$.address.$errors[0].$message
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
                    @click="updateLocation()"
                    class="btn btn-success"
                >
                    Update Now
                </button>
            </div>
        </div>
    </form>
</template>
<script>
import Navbar from "@/components/Header/Navbar.vue";
import useVuelidate from "@vuelidate/core";
import { required, minLength } from "@vuelidate/validators";
import axios from "axios";
import { reactive, computed } from "vue";
import { mapActions } from "vuex";
export default {
    name: "UpdateLocation",
    setup() {
        const state = reactive({
            name: "",
            phone: "",
            address: "",
        });
        const rules = computed(() => {
            return {
                name: { required, minLength: minLength(10) },
                phone: { required, minLength: minLength(10) },
                address: { required, minLength: minLength(10) },
            };
        });
        const v$ = useVuelidate(rules, state);
        return {
            state,
            v$,
        };
    },
    data() {
        return {
            locationId: "",
            userId: "",
            successMessage: "",
            errorMessage: "",
        };
    },
    mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.userId = JSON.parse(user).id;
            this.locationId = this.$route.params.locationId;
            this.canCurrentUserAccessThisLocation();
        }
    },
    components: {
        Navbar,
    },
    methods: {
        ...mapActions(["redirectTo"]),
        async canCurrentUserAccessThisLocation() {
            let result = await axios.get(
                `http://localhost:3000/locations?id=${this.locationId}&userId=${this.userId}`
            );
            if (result.status == 200 && result.data.length > 0) {
                this.locationData = result.data;
                this.state.name = this.locationData[0].name;
                this.state.phone = this.locationData[0].phone;
                this.state.address = this.locationData[0].address;
            } else {
                this.redirectTo({ val: "Home" });
            }
        },
        async updateLocation() {
            this.v$.$validate();
            if (!this.v$.$error) {
                console.log("Validated");
                let results = await axios.put(
                    `http://localhost:3000/locations/${this.locationId}`,
                    {
                        name: this.state.name,
                        phone: this.state.phone,
                        address: this.state.address,
                        userId: this.userId,
                    }
                );
                if (results.status == 200) {
                    this.errorMessage = "";
                    this.successMessage = "Location is updated ...";
                    setTimeout(() => {
                        this.redirectTo({ val: "Home" });
                    }, 2000);
                    console.log("Updated This Location Successfully");
                } else {
                    this.successMessage = "";
                    this.errorMessage =
                        "Something Went Wrong , please try again!";
                    console.log("Failed Adding New Location");
                }
            } else {
                this.successMessage = "";
                this.errorMessage = "You must fill in All required fields!";
            }
        },
    },
};
</script>
<style></style>
