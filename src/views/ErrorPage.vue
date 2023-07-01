<template>
    <div>
        <div class="text-center">
            <RouterLink :to="{ name: 'Home' }">
                <button class="btn btn-info">Visit Home Page</button>
            </RouterLink>
        </div>
        <br />
        <img
            src="../assets/page-not-found.jpg"
            class="rounded mx-auto d-block h400"
        />
    </div>
</template>
<script>
export default {
    name: "ErrorPage",
};
</script>
<style>
.h400 {
    height: 400px;
}
</style>
