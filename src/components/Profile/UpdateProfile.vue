<template>
    <div class="container">
        <Navbar />
        <form @click.prevent>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <h1>Update Your Profile</h1>
                </div>
            </div>
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <input
                        type="text"
                        class="form-control"
                        placeholder="Enter Your Name"
                        v-model="name"
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
                        type="email"
                        class="form-control"
                        placeholder="Enter Your Email"
                        v-model="email"
                    />
                    <span class="error-feedback" v-if="v$.email.$error">{{
                        v$.email.$errors[0].$message
                    }}</span>
                </div>
            </div>
            <br />
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <input
                        type="password"
                        class="form-control"
                        placeholder="Enter Your password"
                        v-model="pass"
                    />
                    <span class="error-feedback" v-if="v$.pass.$error">{{
                        v$.pass.$errors[0].$message
                    }}</span>
                </div>
            </div>
            <br />
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto">
                    <button
                        type="submit"
                        class="btn btn-primary"
                        @click="updateProfileNow()"
                    >
                        Update Profile Now
                    </button>
                </div>
            </div>
            <br />
            <div class="row g-3 align-items-center">
                <div class="col-auto d-block mx-auto error-feedback">
                    {{ updateErr }}
                </div>
            </div>
        </form>
    </div>
</template>
<script>
import Navbar from "@/components/Header/Navbar.vue";
import useVuelidate from "@vuelidate/core";
import { required, email, minLength } from "@vuelidate/validators";
import axios from "axios";
import { mapActions } from "vuex";
export default {
    name: "ProfileForm",
    components: {
        Navbar,
    },
    data() {
        return {
            v$: useVuelidate(),
            name: "",
            email: "",
            pass: "",
            userId: "",
            updateErr: "",
        };
    },
    validations() {
        return {
            name: { required, minLength: minLength(10) },
            email: { required, email },
            pass: { required, minLength: minLength(10) },
        };
    },
    mounted() {
        let user = localStorage.getItem("user-info");
        if (user) {
            this.name = JSON.parse(user).name;
            this.email = JSON.parse(user).email;
            this.pass = JSON.parse(user).pass;
            this.userId = JSON.parse(user).id;
        } else {
            this.redirectTo({ val: "Signup" });
        }
    },
    methods: {
        ...mapActions(["redirectTo"]),
        async updateProfileNow() {
            this.v$.$validate();
            if (!this.v$.$error) {
                console.log("Validated");
                // eslint-disable-next-line no-unused-vars
                let result = await axios.put(
                    `http://localhost:3000/users/${this.userId}`,
                    {
                        name: this.name,
                        email: this.email,
                        pass: this.pass,
                    }
                );
                console.log(result);
                if (result.status == 200) {
                    console.log("Profile Is Updated Successfully");
                    localStorage.setItem(
                        "user-info",
                        JSON.stringify(result.data)
                    );
                    this.redirectTo({ val: "Profile" });
                } else {
                    this.userNotFoundErr = "Something went wrong, try again!";
                }
            } else {
                this.userNotFoundErr =
                    "Something went wrong, refresh the page!";
            }
        },
    },
};
</script>
<style></style>
