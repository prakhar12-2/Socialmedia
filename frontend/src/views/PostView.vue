<template>
    <div class="max-w-7xl mx-auto grid grid-cols-4 gap-4">
        
        <div class="main-center col-span-3 space-y-4">
                    

                    <div class="p-4 bg-white border border-gray-200 rounded-lg"
                    v-if="post.id"
                    >
                        <div class="mb-6 flex items-center justify-between">
                            <div class="flex items-center space-x-6">
                                <img :src="post.created_by.get_avatar" class="w-[40px] rounded-full">
                                
                                <p><strong>{{ post.created_by.name }}</strong></p>
                            </div>

                            <p class="text-gray-600">{{ post.created_at_formatted }} ago</p>
                        </div>
                        <template v-if="post.attachments.length">
                            <img v-for="image in post.attachments" v-bind:key="image.id" :src="image.get_image" class="w-full mb-4 rounded-xl">

                        </template>
                        <p>
                            {{ post.body }}
                        </p>

                        <div class="my-6 flex justify-between">
                            <div class="flex space-x-6" >
                                <div class="flex items-center space-x-2" @click="LikePost(post.id)">
                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                        <path stroke-linecap="round" stroke-linejoin="round" d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z" />
                                    </svg>  
                                    
                                    <span class="text-gray-500 text-xs">{{ post.like_count }} likes</span>
                                </div> 
                                
                                <div class="flex items-center space-x-2">
                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                        <path stroke-linecap="round" stroke-linejoin="round" d="M12 20.25c4.97 0 9-3.694 9-8.25s-4.03-8.25-9-8.25S3 7.444 3 12c0 2.104.859 4.023 2.273 5.48.432.447.74 1.04.586 1.641a4.483 4.483 0 01-.923 1.785A5.969 5.969 0 006 21c1.282 0 2.47-.402 3.445-1.087.81.22 1.668.337 2.555.337z" />
                                    </svg> 

                                    <RouterLink :to="{ name:'postview', params:{id:post.id}}" class="text-gray-500 text-xs">{{ post.comment_count }} comments</RouterLink>
                                </div>
                            </div>
                            
                            <div>
                                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                    <path stroke-linecap="round" stroke-linejoin="round" d="M12 6.75a.75.75 0 110-1.5.75.75 0 010 1.5zM12 12.75a.75.75 0 110-1.5.75.75 0 010 1.5zM12 18.75a.75.75 0 110-1.5.75.75 0 010 1.5z" />
                                </svg>   
                            </div>   
                        </div>  
                    </div>
                    <div class="p-4 ml-6 bg-white border border-gray-200 rounded-xl"
                     v-for="comment in post.comment"
                     v-bind:key=comment.id
                    >
                    <CommentItem v-bind:comment="comment"/>
                    </div>
                    <div class="bg-white border border-gray-200 rounded-lg">
                    <form   v-on:submit.prevent="submitFirm" method="post">
                        <div class="p-4"> 
                            <textarea v-model="body" class="p-4 w-full bg-gray-100 rounded-lg" placeholder="Make a comment to this post"></textarea>
                        </div>

                        <div class="p-4 border-t border-gray-100 flex justify-between">

                            <button class="inline-block py-4 px-6 bg-purple-600 text-white rounded-lg">Comment</button>
                        </div>
                    </form>
                    </div>
        </div>
        <div class="main-right col-span-1 space-y-4">
         <youmknow/>
         <Trends/>
        </div>
    </div>
</template>

<script>
import youmknow from '@/components/youmknow.vue';
import Trends from '@/components/Trends.vue';
import axios from 'axios';
import { RouterLink } from 'vue-router';
import CommentItem from '@/components/CommentItem.vue';

export default{
    name:'PostView',
    components:{
    youmknow,
    Trends,
    RouterLink,
    CommentItem
},
    data(){
        return{ 
            post:{
                post_like:0,
                comment_count:0,
                id:null,
                comment:[],
            },
           //comments:[],
            body:''
        }
    },
    mounted(){
        this.getPost()
    },
    methods:{
        getPost(){
            axios
            .get(`/api/post/${this.$route.params.id}/`)
            .then(response=>{
                console.log('data',response)

                this.post=response.data.post
            })
            .catch(error=>{
                console.log('error',error)
            })
        },
        submitFirm(){
            console.log('submitFirm',this.body)
            axios
            .post(`/api/post/comment/${this.$route.params.id}/`,{
                'body':this.body
            },{
                headers: {
                    'Content-Type': 'application/json',
                 }
                }
            )
            .then(response=>{
                console.log('data',response.data)

               // this.post.comments.unshift(response.data)
                console.log(this.post.comment)
                this.post.comment.unshift(response.data)
                this.comment=''
                this.post.comment_count=this.post.comment_count+1
                //console.log(this.post)
                this.body=''
            })
            .catch(error=>{
                console.log('error',error)
            })
        },
        LikePost(id){
            axios
            .post(`/api/post/like/${id}/`)
            .then(response=>{
                console.log('data',response)
                this.post.like_count=this.post.like_count+1
            })
            .catch(error=>{
                console.log('error',error)
            })
        }

        
    }
}
</script>