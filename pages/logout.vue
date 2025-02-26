<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { Button } from '@/components/ui/button';
import { useRouter } from 'vue-router';
import axios from 'axios';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';

// Use the router for navigation
const router = useRouter();

// Reactive variable to store data
const dataSources = ref([]);

// Fetch data from the API on component mount
onMounted(async () => {
  if (!localStorage.getItem('accessToken')) {
    router.push('/');
    return;
  }

  try {
    const response = await axios.get('https://demo-server.gawx.ai/api/v1/data_sources/', {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('accessToken')}`,
      },
    });
    dataSources.value = response.data;
  } catch (error) {
    console.error('Error fetching data sources:', error);
    alert('Failed to fetch data sources. Please try again later.');
  }

  // Replace the current history state to '/logout'
  window.history.replaceState(null, '', '/logout');
});

// Function to handle logout
const handleLogout = () => {
  // Clear authentication tokens
  localStorage.removeItem('accessToken');
  localStorage.removeItem('refreshToken');

  // Redirect to the login page
  router.push('/');
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
    <Button type="button" @click="handleLogout" class="bg-black hover:bg-gray-500 text-white px-4 py-2 rounded">
      Logout
    </Button>
  </div>
</template>
