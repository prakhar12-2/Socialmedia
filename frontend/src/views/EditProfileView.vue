<template>
    <div class="max-w-7xl mx-auto grid grid-cols-2 gap-4">
        <div class="main-left">
            <div class="p-12 bg-white border border border-gray-200 rounded-lg">
                <h1 class="mb-6 text-2xl">Edit Profile</h1>

                        <p class="mb-6 text-gray-500">
                            Lorem ipsum dolor sit mate. Lorem ipsum dolor sit mate. Lorem ipsum dolor sit mate.
                            Lorem ipsum dolor sit mate. Lorem ipsum dolor sit mate. Lorem ipsum dolor sit mate.
                            <RouterLink to="/profile/editpassword/" class="underline">Change Password</RouterLink>
                        </p>

            </div>
        </div>
        <div class="main-right">
            <div class="p-12 bg-white border border-gray-200 rounded-lg">
                        <form class="space-y-6" v-on:submit.prevent="submitform">
                            <div>
                                <label>E-mail</label><br>
                                <input type="email" v-model="form.email" placeholder="Your e-mail address" class="w-full mt-2 py-4 px-6 border border-gray-200 rounded-lg">
                            </div>
                            <div>
                                <label>Name</label><br>
                                <input type="text" v-model="form.name" placeholder="Your name" class="w-full mt-2 py-4 px-6 border border-gray-200 rounded-lg">
                            </div>
                            <div>
                                <label>Avatar</label><br>
                                <input type="file" ref="file">
                            </div>
                            
                            <template v-if="errors.length>0">
                                <div class="bg-red-300 text-white p-6 rounded-lg">
                                    <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                                </div>
                            </template>
                            <div>
                                <button class="py-4 px-6 bg-purple-600 text-white rounded-lg">Edit</button>
                            </div>
                        </form>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import { useToastStore } from '@/stores/toast'
import { useUserStore } from '@/stores/user'
import { RouterLink } from 'vue-router'

export default {
    setup() {
        const toastStore = useToastStore();
        const userStore = useUserStore();
        return {
            toastStore,
            userStore
        };
    },
    data() {
        return {
            form: {
                email: this.userStore.user.email,
                name: this.userStore.user.name,
            },
            errors: [],
        };
    },
    methods: {
        submitform() {
            this.errors = [];
            if (this.form.email === '') {
                this.errors.push('Your e-mail is missing');
            }
            if (this.form.name === '') {
                this.errors.push('Your name is missing');
            }
            if (this.errors.length === 0) {
                let formdata = new FormData();
                formdata.append('avatar', this.$refs.file.files[0]);
                formdata.append('name', this.form.name);
                formdata.append('email', this.form.email);
                axios
                    .post('/api/editprofile/', formdata, {
                    Headers: {
                        "Content-Type": "multipart/form-data"
                    }
                })
                    .then(response => {
                    if (response.data.message === 'information updated') {
                        this.toastStore.showToast(5000, 'Changed Information updated', 'bg-emerald-500');
                        this.userStore.setUserInfo({
                            id: this.userStore.user.id,
                            email: this.form.email,
                            name: this.form.name,
                            avatar: response.data.user.get_avatar
                        });
                        this.$router.back();
                    }
                    else {
                        this.toastStore.showToast(5000, `${response.data.message}. Please try again`, 'bg-red-300');
                    }
                })
                    .catch(error => {
                    console.log('error', error);
                });
            }
        }
    },
    components: { RouterLink }
}
</script>