<script setup lang="ts">
import { ref, onMounted } from "vue";
import { Button } from "@/components/ui/button";
import { useRouter } from "vue-router";
import axios from "axios";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";

// Define the type for data sources
interface DataSource {
  id: string; // Assuming id is a string, change to number if needed
  name: string;
  type: string;
}

// Router for navigation
const router = useRouter();

// Store data with a defined type
const dataSources = ref<DataSource[]>([]);

// Fetch data or API components
onMounted(async () => {
  if (!localStorage.getItem("accessToken")) {
    router.push("/");
    return;
  }

  try {
    const response = await axios.get(
      "https://demo-server.gawx.ai/api/v1/data_sources/",
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("accessToken")}`,
        },
      }
    );
    dataSources.value = response.data;
  } catch (error) {
    console.error("Error fetching data sources:", error);
    alert("Failed to fetch data sources. Please try again later.");
  }

  // Replace '/logout'
  window.history.replaceState(null, "", "/logout");
});

// Handle logout
const handleLogout = () => {
  localStorage.removeItem("accessToken");
  localStorage.removeItem("refreshToken");

  router.push("/");
};
</script>

<template>
  <div class="mx-auto max-w-screen-xl p-8">
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>ID</TableHead>
          <TableHead>Name</TableHead>
          <TableHead>Type</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        <TableRow v-for="dataSource in dataSources" :key="dataSource.id">
          <TableCell class="font-medium">{{ dataSource.id }}</TableCell>
          <TableCell>{{ dataSource.name }}</TableCell>
          <TableCell>{{ dataSource.type }}</TableCell>
        </TableRow>
      </TableBody>
    </Table>
    <Button
      type="button"
      @click="handleLogout"
      class="bg-black hover:bg-gray-500 text-white px-4 py-2 rounded"
    >
      Logout
    </Button>
  </div>
</template>
