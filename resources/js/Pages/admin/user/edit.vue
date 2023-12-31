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
        to: route('user.user.editpage', {'id': props.additional.user.id})
    },
]
const isLoading = ref(false)
const backActive = ref(false)
const form = ref({
    name: props.additional.user.name,
    email: props.additional.user.email,
    permissions: []
})
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
    // Formatting Permission List
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

    // Init current permission on role
    let userPermissions = form.value.permissions.filter(function (item) {
        return props.additional.user_has_permissions.includes(item.id);
    })
    userPermissions.forEach(function (part, index) {
        let permission = form.value.permissions.find(e => e.id === part.id)
        permission.status = true
    }, userPermissions)
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
    axios.put(route('user.user.updateuser', {'id': props.additional.user.id}), form.value)
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
        <h1 class="text-2xl md:text-3xl text-slate-800 font-bold">Edit Pengguna</h1>
    </div>
    <div class="bg-white shadow-lg rounded-sm border border-slate-200">
        <!-- Panel Content -->
        <div class="p-6 space-y-6">
            <!-- Basic Information -->
            <section>
                <div class="w-full">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">Form Edit Pengguna</h3>
                    <div class="text-sm text-slate-500 mb-4">Sesuaikan Dengan Data Yang Anda Miliki</div>
                    <VInput placeholder="Insert Name" label="Username" :required="true" v-model="form.name"
                        :errorMessage="formError.name" @update:modelValue="formError.name = ''" class="w-1/2"/>

                    <VInput placeholder="Insert Email" label="Email" :required="true" v-model="form.email"
                        :errorMessage="formError.email" @update:modelValue="formError.email = ''" class="w-1/2"/>


                </div>
            </section>

            <section class="border border-slate-200"></section>

            <!-- List Permission -->
            <section>
                <div class="w-full">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">List Permission</h3>
                    <div class="text-sm text-slate-500">List to set the permissions used for the user</div>
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
                        <VButton :is-loading="isLoading" label="Edit" type="primary" @click="submit" />
                    </div>
                </div>
            </footer>
        </div>
    </div>
    <VModalForm :data="itemSelected" :open-dialog="openModalForm" @close="closeModalForm" @successSubmit="successSubmit" :additional="form"/>
</template>