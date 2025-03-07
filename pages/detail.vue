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

// Define the data sources
interface DataSource {
  id: string;
  name: string;
  type: string;
}
// Router for navigation
const router = useRouter();

// Store data with a defined type
const dataSources = ref<DataSource[]>([]);

// control visibility
const showInputBox = ref(false);

const itemName = ref("");
const itemRule = ref("");
const itemDescription = ref("");

interface ItemData {
  item: string;
  name: string;
  rule: string;
  description: string;
}

// Update the itemsData ref declaration
const itemsData = ref<ItemData[]>([]);

const handleFinalSubmit = () => {
  if (itemsData.value.length === 0) {
    alert("Please save some items first");
    return;
  }
  console.log(JSON.stringify(itemsData.value, null, 2));
};

const handleSubmit = () => {
  if (itemsData.value.length === 0) {
    const initialData = Array.from({ length: 5 }, (_, i) => ({
      item: `itme${i + 1}`,
      name: ``,
      rule: ``,
      description: ``,
    }));
    itemsData.value = initialData;
  }

  // Update the current item's data
  const currentIndex = parseInt(currentItem.value) - 1;
  if (currentIndex >= 0 && currentIndex < 5) {
    if (currentIndex !== 0) {
      const item1Data = itemsData.value[0];
      if (
        itemName.value === (item1Data as { name: string }).name ||
        itemRule.value === (item1Data as { rule: string }).rule ||
        itemDescription.value ===
          (item1Data as { description: string }).description
      ) {
        alert("Values for item1 cannot be reused in other items.");
        return;
      }
    }

    itemsData.value[currentIndex] = {
      item: `itme${currentItem.value}`,
      name: itemName.value || `${currentItem.value}`,
      rule: itemRule.value || `${currentItem.value}`,
      description: itemDescription.value || `${currentItem.value}`,
    };
  }

  // Clear form return to list
  itemName.value = "";
  itemRule.value = "";
  itemDescription.value = "";
  showInputBox.value = false;
};

const currentItem = ref("");

// Toggle input box visibility and items
const toggleInputBox = () => {
  showInputBox.value = !showInputBox.value;
};


// Update current item name
const updateCurrentItem = (itemName: string) => {
  currentItem.value = itemName;
};
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
  // window.history.replaceState(null, "", "/detail");
});

// Handle logout
const handleDetail = () => {
  localStorage.removeItem("accessToken");
  localStorage.removeItem("refreshToken");

  router.push("/");
};


</script>

<template>
  <div class="flex flex-col min-h-screen">
    <div class="flex gap-8 flex-1 p-8">
      <!-- Left form -->
      <div class="bg-white w-80 p-4 shadow-md rounded">
        <div class="flex items-center mb-4">
          <button
            v-if="showInputBox"
            class="text-lg mr-4"
            @click="toggleInputBox"
          >
            ←
          </button>
          <h3 class="text-xl font-bold">
            {{ showInputBox ? `item ${currentItem}` : "List of items" }}
          </h3>
        </div>
        <!-- List view -->
        <!-- Update the List -->
        <div v-if="!showInputBox">
          <p 
            class="mt-2 text-lg cursor-pointer hover:bg-gray-100 p-2"
            v-for="n in 5"
            :key="n"
            @click="
              () => {
                updateCurrentItem(n.toString());
                toggleInputBox();
              }
            " 
          >
           items {{ n }}
          </p>
          <button
            @click="handleFinalSubmit"
            class="w-full bg-black hover:bg-gray-600 text-white px-4 py-2 rounded mt-4"
          >
            Submit
          </button>
        </div>
        <!-- Input form -->
        <div v-if="showInputBox" class="space-y-4">
          <div>
            <p class="mb-2">name</p>
            <input
              v-model="itemName"
              type="text"
              class="w-full p-2 border rounded"
              placeholder="Enter text name"
            />
          </div>
          <div>
            <p class="mb-2">rule</p>
            <input
              v-model="itemRule"
              type="text"
              class="w-full p-2 border rounded"
              placeholder="Enter rule"
            />
          </div>
          <div>
            <p class="mb-2">description</p>
            <textarea
              v-model="itemDescription"
              rows="4"
              class="w-full p-2 border rounded"
              placeholder="Enter description"
            />
          </div>
          <button
            @click="handleSubmit"
            class="w-full bg-black hover:bg-gray-600 text-white px-4 py-2 rounded mt-4"
          >
            Save
          </button>
        </div>
      </div>
      <!-- Right table -->
      <div class="bg-white p-4 shadow-md rounded flex-1">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead class="w-[200px]">ID</TableHead>
              <TableHead class="w-[400px]">Name</TableHead>
              <TableHead class="w-[100px]">Type</TableHead>
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
      </div>
    </div>

    <!-- Logout button -->
    <div class="p-4 border-t">
      <Button
        type="button"
        @click="handleDetail"
        class="bg-black hover:bg-gray-700 text-white px-6 py-2 rounded"
      >
        Logout
      </Button>
    </div>
  </div>
</template>
