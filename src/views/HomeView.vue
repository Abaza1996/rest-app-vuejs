<!-- eslint-disable prettier/prettier -->
<template>
    <div class="container">
        <Navbar />
        <p class="text-end">
            Welcome {{ userName }}
            <routerLink :to="{ name: 'Profile' }">
                <button class="btn btn-info" type="button">Profile</button>
            </routerLink>
        </p>
        <routerLink :to="{ name: 'AddNewLocation' }">
            <button type="button" class="btn btn-primary">
                Add New Restaurant
            </button>
        </routerLink>
        <UserLocations :allLocations="listOflocations" />
    </div>
</template>
<script>
import Navbar from "@/components/Header/Navbar.vue";
import UserLocations from "@/components/Locations/UserLocations.vue";
import { mapActions } from "vuex";
import axios from "axios";

export default {
    // eslint-disable-next-line vue/multi-word-component-names
    name: "Home",
    data() {
        return {
            userName: "",
            userId: "",
            listOflocations: [],
        };
    },
    components: {
        Navbar,
        UserLocations,
    },
    async mounted() {
        let user = localStorage.getItem("user-info");
        if (!user) {
            this.redirectTo({ val: "Signup" });
        } else {
            this.userName = JSON.parse(user).name;
            this.userId = JSON.parse(user).id;
            let result = await axios.get(
                `http://localhost:3000/locations?userId=${this.userId}`
            );
            if (result.status == 200 && result.data.length > 0) {
                console.log(result.data);
                this.listOflocations = result.data;
            }
        }
    },
    methods: {
        ...mapActions(["redirectTo"]),
    },
};
</script>
