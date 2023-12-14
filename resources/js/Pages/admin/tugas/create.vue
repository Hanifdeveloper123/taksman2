<script>
export default {
    layout: AppLayout,
};
</script>

<script setup>
import axios from "axios";
import { notify } from "notiwind";
import { ref, onMounted } from "vue";
import { object, string } from "vue-types";
import { Inertia } from "@inertiajs/inertia";
import { Head, Link } from "@inertiajs/inertia-vue3";
import AppLayout from "@/layouts/apps.vue";
import VInput from "@/components/VInput/index.vue";
import VButton from "@/components/VButton/index.vue";
import VBreadcrumb from "@/components/VBreadcrumb/index.vue";
import VBack from "@/components/src/icons/VBack.vue";
import VModalForm from "./CreateModalForm.vue";

const breadcrumb = [
    {
        name: "Dashboard",
        active: false,
        to: route("dashboard.index"),
    },
    {
        name: "Tugas Management",
        active: true,
        to: route("tugas.tugas.createpage"),
    },
];
const isLoading = ref(false);
const backActive = ref(false);
const form = ref({
    judul: "",
    deskripsi: "",
    user_id: "",
    status: "",
    deadline: "",
    permissions: [],
});
const formError = ref({});
const itemSelected = ref({});
const openModalForm = ref(false);

const props = defineProps({
    title: string(),
    additional: object(),
});

const handleActiveAll = (sub_group, index) => {
    const selectId = sub_group[0]["group"] + "_" + index;
    if (document.getElementById(selectId).checked) {
        Object.values(sub_group).forEach((val) => {
            let permission = form.value.permissions.find(
                (e) => e.id === val.id
            );
            permission.status = true;
        });
    } else {
        Object.values(sub_group).forEach((val) => {
            let permission = form.value.permissions.find(
                (e) => e.id === val.id
            );
            permission.status = false;
        });
    }
};

const initData = () => {
    Object.values(props.additional.permission_list).forEach((val) => {
        Object.values(val).forEach((value) => {
            Object.values(value).forEach((res) => {
                const obj = {
                    id: res.id,
                    status: false,
                };
                form.value.permissions.push(obj);
            });
        });
    });
};



const closeModalForm = () => {
    openModalForm.value = false;
};

const successSubmit = (data) => {
    form.value = data;
    openModalForm.value = false;
};



const discard = () => {
    form.value.permissions = [];
    initData();
};
const user_id = ref(null);
fetch('http://127.0.0.1:8000/api/getUser')
    .then((response) => response.json())
    .then((data) => (user_id.value = data));

const submit = async () => {
    isLoading.value = true;
    axios
        .post(route("task.store"), form.value)
        .then((res) => {
            discard();
            notify(
                {
                    type: "success",
                    group: "top",
                },
                2500
            );
            Inertia.visit(route("tugas.tugas.index"));
        })
        .catch((res) => {
            // Handle validation errors
            const result = res.response.data;
            const metaError = res.response.data.meta?.error;
            if (result.hasOwnProperty("errors")) {
                formError.value = ref({});
                Object.keys(result.errors).map((key) => {
                    formError.value[key] = result.errors[key].toString();
                });
            }

            if (metaError) {
                notify(
                    {
                        type: "error",
                        group: "top",
                        text: metaError,
                    },
                    2500
                );
            } else {
                notify(
                    {
                        type: "error",
                        group: "top",
                        text: result.message,
                    },
                    2500
                );
            }
        })
        .finally(() => (isLoading.value = false));
};

onMounted(() => {
    initData();
});
</script>

<template>
    <Head :title="title"></Head>
    <VBreadcrumb :routes="breadcrumb" /> 
    <div class="mb-4 sm:mb-6 flex justify-start space-x-2 items-center">
        <Link :href="route('tugas.tugas.index')" class="cursor-pointer">
            <VBack
                width="32"
                height="32"
                :is-active="backActive"
                @mouseover="backActive = true"
                @mouseout="backActive = false"
            />
        </Link>
        <h1 class="text-2xl md:text-3xl text-slate-800 font-bold">
            Tambah Tugas
        </h1>
    </div>
    <div class="bg-white shadow-lg rounded-sm border border-slate-200">
        <!-- Panel Content -->
        <div class="p-6 space-y-6">
            <!-- Basic Information -->
            <section>
                <div class="w-full">
                    <h3
                        class="text-xl leading-snug text-slate-800 font-bold mb-1"
                    >
                        Form Tambah Tugas
                    </h3>
                    <div class="text-sm text-slate-500 mb-4">
                        Silahkan Isi Form Berikut
                    </div>
                    <VInput
                        placeholder="Masukkan Judul Tugas"
                        label="Judul Tugas"
                        :required="true"
                        v-model="form.judul"
                        :errorMessage="formError.judul"
                        @update:modelValue="formError.judul = ''"
                        class="lg:w-full md:w-auto"
                    />

                    <VInput
                        placeholder="Masukkan Deskripsi Tugas"
                        label="Deskripsi Tugas"
                        :required="true"
                        v-model="form.deskripsi"
                        :errorMessage="formError.deskripsi"
                        @update:modelValue="formError.deskripsi = ''"
                        class="lg:w-full sm:w-auto"
                    />

                <label
                    class="block mb-2 text-sm font-medium  dark:text-white text-slate-500"
                    >User ID</label
                >
                <select
                v-model="form.user_id"
                :required="true"
              
                id="formBookImage"
                    class="bg-gray-50 border border-gray-300  text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                >
                <option v-for="user in user_id" :value="user.id">
                    {{ user.name }}
                </option>
                </select>


                    <label
                        class="block mb-2 text-sm font-medium  dark:text-white text-slate-500" 
                        >Status</label
                    >
                    <select
                    :required="true"
                    v-model="form.status"
                    :errorMessage="formError.status"
                    @update:modelValue="formError.status = ''"
                    id="formBookImage"
                        class="bg-gray-50 border border-gray-300  text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                    >
                    <option value="draft">Draft</option>
                    <option value="on_progress">On Progress</option>
                    <option value="done">Done</option>
                    </select>

                    <VInput
                        placeholder="Insert Deadline"
                        label="Deadline Tugas"
                        :required="true"
                        v-model="form.deadline"
                        type="date"
                        :errorMessage="formError.deadline"
                        @update:modelValue="formError.deadline = ''"
                        class="lg:w-full sm:w-auto"
                    />
                </div>
            </section>

            <!-- Panel footer -->
            <footer>
                <div class="flex flex-col px-6 py-3 border-t border-slate-200">
                    <div class="flex self-end space-x-3">
                        <VButton
                            :is-loading="isLoading"
                            label="Discard"
                            type="default"
                            @click="discard"
                        />
                        <VButton
                            :is-loading="isLoading"
                            label="Kirim"
                            type="primary"
                            @click="submit"
                        />
                    </div>
                </div>
            </footer>
        </div>
    </div>
    <VModalForm
        :data="itemSelected"
        :open-dialog="openModalForm"
        @close="closeModalForm"
        @successSubmit="successSubmit"
        :additional="form"
    />
</template>

<!-- <script>
    import { ref } from 'vue';
    
</script> -->
