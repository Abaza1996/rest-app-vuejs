<template>
    <ul id="nav">
        <routerLink
            tag="li"
            :to="{ name: 'Home', params: { pageTitle: 'HomePage' } }"
            exact
        >
            <a href="">Home</a>
        </routerLink>
        <a href="#" @click="logMeout()">Logout</a>
    </ul>
</template>

<script>
import { mapActions } from "vuex";
export default {
    // eslint-disable-next-line vue/multi-word-component-names
    name: "Navbar",
    methods: {
        ...mapActions(["redirectTo"]),
        logMeout() {
            localStorage.clear();
            // this.$router.push({ name: "Login" });
            this.redirectTo({ val: "Login" });
        },
    },
};
</script>
<style lang="scss" scoped>
$fontColor: darkcyan;
$navColor: #525a5a;
ul {
    display: flex;
    justify-content: center;
    list-style: none;
    margin-bottom: 50px;
    a {
        text-decoration: none;
        font-weight: bold;
        color: $fontColor;
        padding: 10px;
    }
}
a.router-link-exact-active a,
ul a a:hover {
    color: $navColor;
    box-shadow: 2px 2px 4px 0 $navColor;
}
</style>
