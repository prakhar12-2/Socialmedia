<template>
    <div class="max-w-7xl mx-auto grid grid-cols-4 gap-4">
        <div class="main-left col-span-1">
                    <div class="p-4 bg-white border border-gray-200 text-center rounded-lg">
                        <img :src="user.get_avatar" class="mb-6 rounded-full">
                        
                        <p><strong>{{ user.name }}</strong></p>

                        <div class="mt-6 flex space-x-8 justify-around">
                            <RouterLink :to="{name:'friends' ,params:{id:user.id}}" class="text-xs text-gray-500">{{ user.friends_count }} friends</RouterLink>
                            <p class="text-xs text-gray-500">{{ user.posts_count }} posts</p>
                        </div>
                        <div class="mt-6">
                            <button v-if="userStore.user.id!=user.id && can_send_friend_request" class="inline-vlock py-4 px-6 bg-red-600 text-sm text-white rounded-lg" @click="SendRequest">Request</button>
                            <button v-if="userStore.user.id!=user.id" class="mt-4 inline-vlock py-4 px-6 bg-red-600 text-sm text-white rounded-lg" @click="StartConversation">Start Conversation</button>
                            <button  v-if="userStore.user.id == user.id" class="inline-block mr-6 py-4 px-6 bg-purple-600 text-sm text-white rounded-lg" @click="Logout">Logout</button>
                            <RouterLink to="/profile/edit/"  v-if="userStore.user.id == user.id" class="inline-block py-4 px-6 bg-emerald-600 text-sm text-white rounded-lg">Edit profile</RouterLink>
                        </div>
                    </div>
        </div>
        <div class="main-center col-span-2 space-y-4">
                    <div 
                    class="bg-white border border-gray-200 rounded-lg"
                    v-if="userStore.user.id === user.id">
                    <form   v-on:submit.prevent="submitFirm" method="post">
                        <div class="p-4"> 
                            <textarea v-model="body" class="p-4 w-full bg-gray-100 rounded-lg" placeholder="What are you thinking about?"></textarea>
                            <!--label>
                                <input type="checkbox" v-model="is_private"> Private
                            </label-->
                            <div id="preview" v-if="url">
                                <img :src="url" class="w-[100px] rounded-xl">
                            </div>
                        </div>

                        <div class="p-4 border-t border-gray-100 flex justify-between">
                            <label class="inline-block py-4 px-6 bg-gray-600 text-white rounded-lg">
                             <input type="file" ref="file" @change="onFileChange">
                             Attach image
                            </label>
                            <button class="inline-block py-4 px-6 bg-purple-600 text-white rounded-lg">Post</button>
                        </div>
                    </form>
                    </div>

                    <div class="p-4 bg-white border border-gray-200 rounded-lg"
                         v-for="post in posts"
                         v-bind:key="post.id">
                         <FeedItem v-bind:post="post"/>
                    </div>
        </div>
        <div class="main-right col-span-1 space-y-4">
         <youmknow/>
         <Trends/>
        </div>
    </div>
</template>
<style>
input[type="file"]{
    display: none;
}
</style>
<script>
import youmknow from '@/components/youmknow.vue';
import Trends from '@/components/Trends.vue';
import axios from 'axios';
import { useUserStore } from '@/stores/user'
import FeedItem from '@/components/FeedItem.vue';
import { RouterLink } from 'vue-router';
import { useToastStore } from '@/stores/toast';
export default{
    name:'FeedView',
    setup() {
        const userStore = useUserStore()
        const toastStore =useToastStore()
        return {
            userStore,
            toastStore
        }
    },
    components:{
    youmknow,
    Trends,
    FeedItem,
    RouterLink
},
    data(){
        return{
            posts:[],
            user:{
                id:null
            },
            body:'',
            url:'',
            can_send_friend_request:null,
            is_private:false
        }
    },
    mounted(){
        this.getfeed()
    },
    watch:{
        '$route.params.id':{
            handler:function(){
                this.getfeed()
            },
            deep:true,
            immediate:true
        }
    }
    ,
    methods:{
        onFileChange(e){
            const file=e.target.files[0]
            this.url=URL.createObjectURL(file)
        },
        StartConversation(){
            console.log('Start of a new conversation')
            axios
            .get(`api/chat/${this.$route.params.id}/get_or_start/`)
            .then(response=>{
                console.log('data',response.data)
                this.$router.push('/chat')
            })
            .catch(error=>{
                console.log('error',error)
            })
        },
        SendRequest(){
            axios
            .post(`api/friends/request/${this.$route.params.id}/`)
            
            .then(response=>{
                console.log('data',response)
                this.can_send_friend_request=false
                if(response.data.message=='Request Sent successfully'){
                    this.toastStore.showToast(5000, 'Request Sent Successfully', 'bg-emerald-500')
                }
                else{
                    this.toastStore.showToast(5000, 'Request has been sent already', 'bg-red-300')
                }
            })
            .catch(error=>{
                console.log('error:',error)
            })
        },
        getfeed(){
            axios
            .get(`/api/post/profile/${this.$route.params.id}/`)
            .then(response=>{
                console.log('data',response)
                this.posts=response.data.posts
                this.user=response.data.user
                this.can_send_friend_request=response.data.can_send_friend_request
               // console.log('http://127.0.0.1:8000'+this.user.avatar)
            })
            .catch(error=>{
                console.log('error',error)
            })
        },
        submitFirm(){
            console.log('submitFirm',this.body)
            let formdata = new FormData();
            formdata.append('image', this.$refs.file.files[0]);
            formdata.append('body', this.body);
            formdata.append('is_private',this.is_private)
            axios
            .post('/api/post/create/',formdata,{
                headers: {
                    "Content-Type" : "multipart/form-data",
                }
                }
            )
            .then(response=>{
                console.log('data',response.data)
                this.posts.unshift(response.data)
                this.user.posts_count=this.user.posts_count+1
                this.is_private=false
                this.body=''
                this.url=null
            })
            .catch(error=>{
                console.log('error',error)
            })
        },
        Logout(){
            this.userStore.removeToken()
            this.$router.push('/login')
            

        }

        
    }
}
</script>