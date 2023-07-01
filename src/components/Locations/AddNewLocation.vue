<template>
    <Navbar />
    <form @click.prevent>
        <div class="row g-3 align-items-center">
            <div class="col-auto d-block mx-auto">
                <h1>Add New Restaurant</h1>
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
                    @click="addLocation()"
                    class="btn btn-primary"
                >
                    Add Now
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
    name: "AddNewLocation",
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
    components: {
        Navbar,
    },
    data() {
        return {
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
        }
    },
    methods: {
        ...mapActions(["redirectTo"]),
        async addLocation() {
            this.v$.$validate();
            if (!this.v$.$error) {
                console.log("Validated");
                let results = await axios.post(
                    `http://localhost:3000/locations`,
                    {
                        name: this.state.name,
                        phone: this.state.phone,
                        address: this.state.address,
                        userId: this.userId,
                    }
                );
                if (results.status == 201) {
                    this.errorMessage = "";
                    this.successMessage = "Added New Location";
                    setTimeout(() => {
                        this.redirectTo({ val: "Home" });
                    }, 2000);
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
