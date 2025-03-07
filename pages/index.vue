<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useForm } from "vee-validate";
import { toTypedSchema } from "@vee-validate/zod";
import * as z from "zod";
import { Button } from "@/components/ui/button";
import {
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import axios from "axios";
import { useRouter } from "vue-router";

// validation schema
const formSchema = toTypedSchema(
  z.object({
    email: z.string().email(),
    password: z.string().min(4),
  })
);

// track login status
const isLoggedIn = ref(false);

//  router or navigation
const router = useRouter();

//  form with validation
const form = useForm({
  validationSchema: formSchema,
});

const handleError = (status: number, data: any) => {
  switch (status) {
    case 400:
      return "Bad Request: Please check your input.";
    case 401:
      return "Unauthorized: Please check your credentials.";
    case 403:
      return "Forbidden: You do not have permission to access this resource.";
    case 404:
      return "Not Found: The requested resource could not be found.";
    case 500:
      return "Server Error: Please try again later.";
    default:
      return `Error: ${status} - ${data}`;
  }
};

// Check login status or commponets
onMounted(() => {
  window.history.replaceState(null, "", "/Login");
  const accessToken = localStorage.getItem("accessToken");
  if (accessToken) {
    isLoggedIn.value = true;
  }
});

const onSubmit = form.handleSubmit(async (values) => {
  try {
    const response = await axios.post(
      "https://demo-server.gawx.ai/api/token/",
      {
        email: values.email,
        password: values.password,
      },
      {
        headers: {
          "Content-Type": "application/json",
        },
      }
    );

    // local storage
    const accessToken = response.data.access;
    const refreshToken = response.data.refresh;
    localStorage.setItem("accessToken", accessToken);
    localStorage.setItem("refreshToken", refreshToken);

    router.push("/detail");
  } catch (error) {
    // Handle errors
    if (axios.isAxiosError(error) && error.response) {
      const errorMessage = handleError(
        error.response.status,
        error.response.data.detail
      );
      alert(errorMessage);
    } else {
      alert("An unexpected error occurred.");
    }
  }
});
</script>

<template>
  <form @submit.prevent="onSubmit" class="space-y-8 w-96 mx-auto mt-40">
    <FormField v-slot="{ componentField }" name="email">
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl>
          <Input
            type="email"
            placeholder="Enter your email"
            v-bind="componentField"
          />
        </FormControl>
        <FormDescription>Your email address.</FormDescription>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="password">
      <FormItem>
        <FormLabel>Password</FormLabel>
        <FormControl>
          <Input
            type="password"
            placeholder="Enter your password"
            v-bind="componentField"
          />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>
    <Button type="submit"> Login </Button>
  </form>
</template>
