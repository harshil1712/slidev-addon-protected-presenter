<script setup lang="ts">
import { onMounted } from "vue";

import { useNav, configs } from "@slidev/client";

const { isPresenter, router, currentPath, currentSlideNo } = useNav();

const props = defineProps({
  passwordUrl: {
    type: String,
    default: "/auth" || configs.auth.passwordUrl,
  },
  staticPassword: {
    type: String,
    default: configs.auth.password || "your-super-strong-static-password",
  },
  // Message to show when prompting for password
  promptMessage: {
    type: String,
    default: "Enter presenter password:",
  },
});

let password = props.staticPassword;

onMounted(async () => {
  // If a password URL is provided, fetch the password
  if (props.passwordUrl) {
    try {
      const response = await fetch(props.passwordUrl);
      if (response.ok) {
        password = (await response.text()).trim();
        console.log("Remote password loaded");
      }
    } catch (error) {
      console.error("Failed to load remote password, using fallback", error);
    }
  }

  const isAuthenticated = localStorage.getItem("slidev-presenter-auth");

  if (isPresenter.value) {
    if (isAuthenticated !== "true") {
      const inputPassword = prompt(props.promptMessage);

      if (!inputPassword) {
        router.push("/" + currentSlideNo.value);
        return;
      }

      if (inputPassword === password) {
        localStorage.setItem("slidev-presenter-auth", "true");
      } else {
        alert("Incorrect password");
        router.push("/" + currentSlideNo.value);
      }
    }
  }
});
</script>

<template></template>
