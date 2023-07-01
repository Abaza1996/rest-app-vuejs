<!-- eslint-disable vue/multi-word-component-names -->
<template>
    <div class="container">
        <SignUpForm />
    </div>
</template>
<script>
import SignUpForm from "@/components/Signup/Signup.vue";
export default {
    // eslint-disable-next-line vue/multi-word-component-names
    name: "Signup",
    components: {
        SignUpForm,
    },
};
</script>
<style></style>
