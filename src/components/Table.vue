<script setup>
import { ref } from 'vue';
import ConfirmModal from './ConfirmModal.vue'
const props = defineProps(['data', 'handleEdit', 'handleDelete'])
const isDeleteModal = ref(false)
const deleteIndex = ref(null)

const handleDeleteModal = (key) => {
    isDeleteModal.value = true
    deleteIndex.value = key
}
const closeDeleteModal = () => {
    isDeleteModal.value = false
}
</script>
<template>
    <div class="container m-auto pb-5" v-if="props.data?.length > 0">
        <h2>User Data</h2>
        <div class="table-responsive">
            <table id="tblOutput" class="table table-striped table-bordered">
                <thead>
                    <tr>
                        <th v-for="(value, key) of data[0]">{{ key }}</th>
                        <th>Edit</th>
                        <th>Delete</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(value, key) of data" :id="`tr${key + 1}`">
                        <td v-for="(val, k) of value">
                            <template v-if="k === 'Hobbies'">{{ val.toString() }}</template>
                            <template v-else>{{ val }}</template>
                        </td>
                        <td><button class="btn btn-success" style="font-size: 14px;" @click="handleEdit(key)">Edit <i
                                    class="bi bi-pencil-fill"></i></button></td>
                        <td><button class="btn btn-danger" style="font-size: 14px;" @click="handleDeleteModal(key)">Delete <i
                            class="bi bi-trash"></i></button></td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
    <ConfirmModal v-if="isDeleteModal" :closeDeleteModal="closeDeleteModal" :handleDelete="handleDelete"
        :deleteIndex="deleteIndex" /></template>