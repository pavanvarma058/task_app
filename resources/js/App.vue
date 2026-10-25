<script setup>
import { ref, onMounted } from "vue";

const tasks = ref([]);
const newTitle = ref("");

async function loadTasks() {
    const res = await fetch("/api/tasks");
    tasks.value = await res.json();
}

async function addTask() {
    if (!newTitle.value.trim()) return;
    await fetch("/api/tasks", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
        },
        body: JSON.stringify({ title: newTitle.value }),
    });
    newTitle.value = "";
    loadTasks();
}

async function toggleTask(task) {
    await fetch(`/api/tasks/${task.id}`, {
        method: "PATCH",
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
        },
        body: JSON.stringify({ completed: !task.completed }),
    });
    loadTasks();
}

async function deleteTask(task) {
    await fetch(`/api/tasks/${task.id}`, { method: "DELETE" });
    loadTasks();
}

onMounted(loadTasks);
</script>

<template>
    <div class="min-h-screen bg-gray-950 py-12 text-gray-100">
        <div
            class="max-w-md mx-auto bg-gray-900 rounded-xl shadow-lg p-6 border border-gray-800"
        >
            <h1 class="text-2xl font-bold mb-4 text-white">My Tasks</h1>

            <form @submit.prevent="addTask" class="flex gap-2 mb-4">
                <input
                    v-model="newTitle"
                    placeholder="New task..."
                    class="flex-1 bg-gray-800 border border-gray-700 text-gray-100 placeholder-gray-500 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <button
                    type="submit"
                    class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-500"
                >
                    Add
                </button>
            </form>

            <p v-if="tasks.length === 0" class="text-gray-500 text-center py-4">
                No tasks yet. Add one above!
            </p>

            <ul class="space-y-2">
                <li
                    v-for="task in tasks"
                    :key="task.id"
                    class="flex items-center gap-3 p-3 bg-gray-800 border border-gray-700 rounded-lg"
                >
                    <input
                        type="checkbox"
                        :checked="task.completed"
                        @change="toggleTask(task)"
                        class="w-4 h-4 accent-blue-500"
                    />
                    <span
                        class="flex-1"
                        :class="{
                            'line-through text-gray-500': task.completed,
                        }"
                    >
                        {{ task.title }}
                    </span>
                    <button
                        @click="deleteTask(task)"
                        class="text-red-400 hover:text-red-300"
                    >
                        ✕
                    </button>
                </li>
            </ul>
        </div>
    </div>
</template>
