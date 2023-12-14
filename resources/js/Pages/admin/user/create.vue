<script>
export default {
    layout: AppLayout
}
</script>
<script setup>
import axios from "axios"
import { notify } from "notiwind"
import { ref, onMounted } from "vue"
import { object, string } from "vue-types"
import { Inertia } from '@inertiajs/inertia'
import { Head, Link } from "@inertiajs/inertia-vue3"
import AppLayout from '@/layouts/apps.vue'
import VInput from "@/components/VInput/index.vue"
import VButton from "@/components/VButton/index.vue"
import VBreadcrumb from '@/components/VBreadcrumb/index.vue'
import VBack from '@/components/src/icons/VBack.vue'
import VModalForm from './CreateModalForm.vue'

const breadcrumb = [
    {
        name: "Dashboard",
        active: false,
        to: route('dashboard.index')
    },
    {
        name: "User Management",
        active: true,
        to: route('user.user.createpage')
    },
]
const isLoading = ref(false)
const backActive = ref(false)
const form = ref({
    name: '',
    email: '',
    password: '',
    permissions: []
})
const show = ref(false);
const formError = ref({})
const itemSelected = ref({})
const openModalForm = ref(false)

const props = defineProps({
    title: string(),
    additional: object()
})

const handleActiveAll = (sub_group, index) => {
    const selectId = sub_group[0]['group'] + '_' + index
    if (document.getElementById(selectId).checked) {
        Object.values(sub_group).forEach(val => {
            let permission = form.value.permissions.find(e => e.id === val.id)
            permission.status = true
        })
    } else {
        Object.values(sub_group).forEach(val => {
            let permission = form.value.permissions.find(e => e.id === val.id)
            permission.status = false
        })
    }
}

const initData = () => {
    Object.values(props.additional.permission_list).forEach(val => {
        Object.values(val).forEach(value => {
            Object.values(value).forEach(res => {
                const obj = {
                    id: res.id,
                    status: false
                }
                form.value.permissions.push(obj)
            })
        })
    })
}

const handleManageModal = (data) => {
    itemSelected.value = data
    openModalForm.value = true
}

const closeModalForm = () => {
    openModalForm.value = false
}

const successSubmit = (data) => {
    form.value = data
    openModalForm.value = false
}

const activeChecker = (subGroup) => {
    return form.value.permissions.filter(function (item) {
        return subGroup.map(function (item) { return item['id'] }).includes(item.id);
    }).map(function (item) { return item['status'] }).includes(false) ? false : true
}

const discard = () => {
    form.value.permissions = []
    initData()
}

const submit = async () => {
    isLoading.value = true
    axios.post(route('user.user.storeuser'), form.value)
        .then((res) => {
            discard()
            notify({
                type: "success",
                group: "top",
                text: res.data.meta.message
            }, 2500)
            Inertia.visit(route('user.user.index'))
        }).catch((res) => {
            // Handle validation errors
            const result = res.response.data
            const metaError = res.response.data.meta?.error
            if (result.hasOwnProperty('errors')) {
                formError.value = ref({});
                Object.keys(result.errors).map((key) => {
                    formError.value[key] = result.errors[key].toString();
                });
            }

            if (metaError) {
                notify({
                    type: "error",
                    group: "top",
                    text: metaError
                }, 2500)
            } else {
                notify({
                    type: "error",
                    group: "top",
                    text: result.message
                }, 2500)
            }
        }).finally(() => isLoading.value = false)
}

onMounted(() => {
    initData()
});
</script>

<template>
    <Head :title="title"></Head>
    <VBreadcrumb :routes="breadcrumb" />
    <div class="mb-4 sm:mb-6 flex justify-start space-x-2 items-center">
        <Link :href="route('user.user.index')" class="cursor-pointer">
            <VBack width="32" height="32" :is-active="backActive" @mouseover="backActive = true" @mouseout="backActive = false"/>
        </Link>
        <h1 class="text-2xl md:text-3xl text-slate-800 font-bold">Tambah Pengguna</h1>
    </div>
    <div class="bg-white shadow-lg rounded-sm border border-slate-200">
        <!-- Panel Content -->
        <div class="p-6 space-y-6">
            <!-- Basic Information -->
            <section>
                <div class="w-full">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">Form Tambah Pengguna</h3>
                    <div class="text-sm text-slate-500 mb-4">Silahkan Isi Form Berikut</div>
                    <VInput placeholder="Masukkan Nama Anda" label="Name" :required="true" v-model="form.name"
                        :errorMessage="formError.name" @update:modelValue="formError.name = ''" class="lg:w-full sm:w-auto"/>

                    <VInput placeholder="Masukkan Email Anda" label="Email" :required="true" v-model="form.email"
                        :errorMessage="formError.email" @update:modelValue="formError.email = ''" class="lg:w-full sm:w-auto"/>

                        <VInput :type="show ? 'text' : 'password'" placeholder="Password" label="Password" v-model="form.password">
                            <template v-slot:icon  :errorMessage="formError.password" @update:modelValue="formError.password = ''" class="lg:w-full sm:w-auto">
                                <span class="absolute inset-y-0 right-0 flex items-center pr-2 cursor-pointer" @click="show = !show">
                                    <svg v-if="!show" width="24" height="24" viewBox="0 0 12 22" fill="none" xmlns="http://www.w3.org/2000/svg">
                                        <path d="M2.5 7L10 15.25" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                        <path d="M7.51074 12.5123C7.16664 12.8275 6.71645 13.0016 6.2498 12.9998C5.87136 12.9998 5.50179 12.8852 5.18968 12.6712C4.87756 12.4572 4.63751 12.1537 4.50108 11.8008C4.36465 11.4478 4.33823 11.0617 4.42529 10.6935C4.51235 10.3252 4.70882 9.99185 4.98886 9.7373" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                        <path d="M3.71875 8.34082C1.80625 9.30645 1 11.1252 1 11.1252C1 11.1252 2.5 14.5002 6.25 14.5002C7.12871 14.5074 7.99645 14.3049 8.78125 13.9096" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                        <path d="M10.0277 13.0516C11.0496 12.1375 11.4996 11.125 11.4996 11.125C11.4996 11.125 9.99961 7.75001 6.24961 7.75001C5.92451 7.74937 5.59993 7.77603 5.2793 7.8297" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                        <path d="M6.60156 9.2832C7.0003 9.35875 7.36365 9.56195 7.63677 9.86213C7.90989 10.1623 8.07797 10.5432 8.11563 10.9473" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                    </svg>
                                    <svg v-else width="24" height="24" viewBox="0 0 12 22" fill="none" xmlns="http://www.w3.org/2000/svg">
                                        <path d="M6 7.625C2.25 7.625 0.75 11 0.75 11C0.75 11 2.25 14.375 6 14.375C9.75 14.375 11.25 11 11.25 11C11.25 11 9.75 7.625 6 7.625Z" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                        <path d="M6 12.875C7.03553 12.875 7.875 12.0355 7.875 11C7.875 9.96447 7.03553 9.125 6 9.125C4.96447 9.125 4.125 9.96447 4.125 11C4.125 12.0355 4.96447 12.875 6 12.875Z" stroke="#475569" stroke-linecap="round" stroke-linejoin="round" />
                                    </svg>
                                </span>
                            </template>
                        </VInput>
                </div>
            </section>

            <section class="border border-slate-200"></section>

            <!-- List Permission -->
            <section>
                <div class="w-full">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">List Permission</h3>
                    <div class="text-sm text-slate-500">List to set the permissions used for the role</div>
                </div>
                <div class="w-full my-6" v-for="(data, index) in additional.permission_list" :key="index">
                    <p class="uppercase font-bold text-slate-500 text-xl">{{ index.replace(/_/g, " ") }}</p>
                    <div class="w-full flex justify-between my-4 border-b pb-4" v-for="(subGroup, index) in data" :key="index" 
                        :class="{
                            'border-b-0 pb-0' :  Object.keys(data).pop() === index
                        }">
                        <div class="max-w-[60%]">
                            <p class="capitalize text-base text-slate-800 font-semibold">{{ index.replace(/_/g, " ") }}</p>
                            <div class="font-normal text-slate-400 text-sm">
                                (<span v-for="(permission, index) in subGroup" :key="index">
                                    {{ permission.label }}{{index + 1 === subGroup.length ? '' : ', '}}
                                </span>)
                            </div>
                        </div>
                        <div class="flex">
                            <div class="flex  my-auto">
                                <div class="text-sm font-normal text-slate-400 italic mr-2">{{ subGroup.length === 1 ? 'Active' : 'Active All'}}</div>
                                <div class="form-switch">
                                    <input type="checkbox" :id="subGroup[0]['group'] + '_' + index" class="sr-only" @change="handleActiveAll(subGroup, index)"
                                        :checked="activeChecker(subGroup)" />
                                    <label class="bg-slate-400" :for="subGroup[0]['group'] + '_' + index">
                                        <span class="bg-white shadow-sm" aria-hidden="true"></span>
                                        <span class="sr-only">Enable smart sync</span>
                                    </label>
                                </div>
                            </div>
                            <div class="my-auto ml-2" v-if="subGroup.length > 1">
                                <VButton label="Manage" type="outline-primary" @click="handleManageModal(subGroup)" />
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- Panel footer -->
            <footer>
                <div class="flex flex-col px-6 py-3 border-t border-slate-200">
                    <div class="flex self-end space-x-3">
                        <VButton :is-loading="isLoading" label="Discard" type="default" @click="discard" />
                        <VButton :is-loading="isLoading" label="Kirim" type="primary" @click="submit" />
                    </div>
                </div>
            </footer>
        </div>
    </div>
    <VModalForm :data="itemSelected" :open-dialog="openModalForm" @close="closeModalForm" @successSubmit="successSubmit" :additional="form"/>
</template>