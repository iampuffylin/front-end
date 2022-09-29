<template>
    <div class="post-wrap" v-if="post && post.post_type">
        <LiveVideo v-if="post.post_type === 'INSTAGRAM_LIVE' || post.post_type === 'TIKTOK_LIVE'"
        :post="post" 
        :is_likeboard_post="is_likeboard_post" 
        :is_my_likeboard="is_my_likeboard" 
        :likeboard_id="likeboard_id" 
        :is_post_details="is_post_details"
        :display_type="display_type" :post_spec="post_spec" />
        <Post v-else 
        :post="post" 
        :is_likeboard_post="is_likeboard_post" 
        :is_my_likeboard="is_my_likeboard" 
        :likeboard_id="likeboard_id" 
        :is_post_details="is_post_details"
        :display_type="display_type" :post_spec="post_spec" :is_connected="is_connected" :is_fan_made="is_fan_made" />
    </div>
</template>

<script>
import LiveVideo from '@/components/post/LiveVideo.vue';
import Post from '@/components/post/Post.vue';

export default {
    name: 'post-wrap',
    props: {
        post: Object,
        is_likeboard_post: Boolean,
        is_my_likeboard: Boolean,
        likeboard_id: {
            type: Number,
            default: 0,
        },
        is_post_details: Boolean,
        display_type: String,
        post_spec: String,
        is_connected: Boolean,
        is_fan_made: Boolean,
    },
    components: {
        Post,
        LiveVideo,
    },

}
</script>
