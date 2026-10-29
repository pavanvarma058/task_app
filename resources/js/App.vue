<script setup>
import { ref, computed, onMounted } from "vue";

const tasks = ref([]);
const newTitle = ref("");
const error = ref("");

async function loadTasks() {
    const res = await fetch("/api/tasks");
    tasks.value = await res.json();
}

async function addTask() {
    error.value = "";

    const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
            "content-type": "application/json",
            Accept: "application/json",
        },
        body: JSON.stringify({ title: newTitle.value }),
    });

    if (!res.ok) {
        const data = await res.json().catch(() => ({}));
        error.value =
            data.errors?.title?.[0] ?? data.message ?? "Something went wrong.";
        return;
    }
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

// --- Editing ---
const editingId = ref(null); // which task is being edited (null = none)
const editTitle = ref(""); // the text in the edit box

// Custom directive: auto-focus an input when it appears
const vFocus = { mounted: (el) => el.focus() };

function startEdit(task) {
    editingId.value = task.id;
    editTitle.value = task.title;
}

function cancelEdit() {
    editingId.value = null;
}

async function saveEdit(task) {
    if (editingId.value !== task.id) return; // already saved or cancelled

    const title = editTitle.value.trim();
    if (!title || title === task.title) {
        cancelEdit();
        return;
    }

    await fetch(`/api/tasks/${task.id}`, {
        method: "PATCH",
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
        },
        body: JSON.stringify({ title }),
    });

    cancelEdit();
    loadTasks();
}

// --- Filtering ---
const filter = ref("all"); // 'all' | 'active' | 'completed'

const filteredTasks = computed(() => {
    if (filter.value === "active")
        return tasks.value.filter((t) => !t.completed);
    if (filter.value === "completed")
        return tasks.value.filter((t) => t.completed);
    return tasks.value;
});

const remaining = computed(
    () => tasks.value.filter((t) => !t.completed).length,
);

onMounted(loadTasks);
</script>

<template>
    <div class="min-h-screen bg-gray-950 py-12 text-gray-100">
        <div
            class="max-w-md mx-auto bg-gray-900 rounded-xl shadow-lg p-6 border border-gray-800"
        >
            <h1 class="text-2xl font-bold mb-4 text-white">My Tasks</h1>

            <p class="text-sm text-gray-500 mb-4 -mt-2">
                Double-click a task to edit it
            </p>

            <form @submit.prevent="addTask" class="flex gap-2 mb-4">
                <input
                    v-model="newTitle"
                    placeholder="New task..."
                    @input="error = ''"
                    class="flex-1 bg-gray-800 border border-gray-700 text-gray-100 placeholder-gray-500 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <button
                    type="submit"
                    class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-500"
                >
                    Add
                </button>
            </form>
            <p v-if="error" class="text-red-400 text-sm -mt-2 mb-4">
                {{ error }}
            </p>
            <p
                v-else-if="filteredTasks.length === 0"
                class="text-gray-500 text-center py-4"
            >
                No {{ filter }} tasks.
            </p>
            <ul class="space-y-2">
                <li
                    v-for="task in filteredTasks"
                    :key="task.id"
                    class="flex items-center gap-3 p-3 bg-gray-800 border border-gray-700 rounded-lg"
                >
                    <input
                        type="checkbox"
                        :checked="task.completed"
                        @change="toggleTask(task)"
                        class="w-4 h-4 accent-blue-500"
                    />
                    <input
                        v-if="editingId === task.id"
                        v-model="editTitle"
                        v-focus
                        @keyup.enter="saveEdit(task)"
                        @keyup.esc="cancelEdit"
                        @blur="saveEdit(task)"
                        class="flex-1 bg-gray-700 border border-blue-500 text-gray-100 rounded px-2 py-1 focus:outline-none"
                    />
                    <span
                        v-else
                        @dblclick="startEdit(task)"
                        class="flex-1 cursor-pointer select-none"
                        :class="{
                            'line-through text-gray-500': task.completed,
                        }"
                        title="Double-click to edit"
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
            <div
                v-if="tasks.length"
                class="flex items-center justify-between mt-4 text-sm text-gray-400"
            >
                <span
                    >{{ remaining }}
                    {{ remaining === 1 ? "task" : "tasks" }} left</span
                >

                <div class="flex gap-1">
                    <button
                        v-for="f in ['all', 'active', 'completed']"
                        :key="f"
                        @click="filter = f"
                        class="px-3 py-1 rounded-md capitalize"
                        :class="
                            filter === f
                                ? 'bg-blue-600 text-white'
                                : 'hover:bg-gray-800'
                        "
                    >
                        {{ f }}
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>
