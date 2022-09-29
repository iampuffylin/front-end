<template>
    <article v-if="post" :id="'js-post--' + post.post_id" :data-post-id="post.post_id" class="new-post" :class="['new-post--' + post.post_type.toLowerCase(), {'scraped-post': post.post_scraped}, {'oc-post': post.is_content}]" v-observe-visibility="{callback: sendPageView, once:true, intersection: {threshold: 0.5}, throttle: 1000}">
        <!-- //////header////// -->
        <header v-if="post_spec==='motom' && display_type==='feed' && (typeof post.post_scraped === 'undefined' || post.post_scraped === false) && !post.is_content" class="new-post--info">
            <div class="new-post--author-info motom">
                <div v-if="post.post_author_id != this.$store.state.user.id">
                    <Avatar :to="`/user/${post.post_author_id}/profile`" :image="post.post_author_image" :type="post.post_author_type" :size="'medium'" :is_influencer="Boolean(post.is_influencer)"/>
                </div>
                <div v-else>
                    <Avatar :to="this.$route.name === 'me' ? '#' : '/me'" :image="post.post_author_image" :type="post.post_author_type" :size="'medium'" :is_influencer="Boolean(post.is_influencer)"/>
                </div>
                <div class="info">
                    <span v-if="post.post_author_type === 2 && !post.post_author_image" class="info--post-author" @click="gotoProfile()">Motom Creator Watch</span>
                    <span v-else class="info--post-author" @click="gotoProfile()">{{post.post_author_name}}</span>
                </div>
                <div v-if="post.is_pin" class="pinned">
                    <svg><use xlink:href="#pinned--sprite"></use></svg>Pinned
                </div>
                <div v-if="$store.state.is_logged" class="new-post--edit-post" @click.prevent="openEditMenu">
                    <svg><use xlink:href="#post-pop-up-menu--sprite"></use></svg>
                </div>
                <div v-if="post.post_caption" class="caption" v-html="captionAddTag"></div>
            </div>
        </header>
        <header v-else-if="post_spec==='motom' && display_type=='grid' && post.is_pin" class="new-post--header--grid">
            <div class="pinned">
                <svg><use xlink:href="#pinned--sprite"></use></svg>Pinned
            </div>
            <div class="unpin" @click="pinPost(false)">Unpin</div>
        </header>
        <header v-else-if="post_spec==='social' && display_type==='feed'" class="new-post--header--feed">
            <div class="to-view-posts" v-if="$route.name === 'private-profile-me' && post.use_in_post && post.in_post.length !== 0" @click="popupPosts()">
                <i><svg><use xlink:href="#checked-circle-2--sprite"></use></svg></i>
                <p>This has been posted to Motom!  </p>
                <span>View <svg><use xlink:href="#arrow-select--sprite"></use></svg></span>
            </div>
            <div class="to-create" @click="createPost()" v-else-if="$route.name === 'private-profile-me'">
                <i><svg><use xlink:href="#create--sprite"></use></svg></i>
                <p>You haven't created a Motom post using this yet. </p>
                <span>Create <svg><use xlink:href="#arrow-select--sprite"></use></svg></span>
            </div>
            <div class="new-post--author-info social">
                <div class="left" @click="openLink">
                    <div :class="[post.post_type.toLowerCase()]">
                        <Icon :type="post.post_type.toLowerCase()" :icon="post.post_type.toLowerCase() + '-no-circle'" :size="'small'" :shape="'circle'" />
                    </div>
                    <div v-if="post.social_username" class="user-id">
                        <span v-if="post.post_type != 'YOUTUBE' && post.post_type != 'TIKTOK'">@</span>{{post.social_username}}
                    </div>
                </div>
                <div class="right">
                    <div v-if="post.is_pin" class="pinned">
                        <svg><use xlink:href="#pinned--sprite"></use></svg>Pinned
                    </div>
                    <div v-if="$store.state.is_logged" class="edit" @click="openEditMenu">
                        <svg><use xlink:href="#post-pop-up-menu--sprite"></use></svg>
                    </div>
                </div>
            </div>
        </header>
        <header v-if="post_spec==='social' && display_type=='grid' && post.is_pin" class="new-post--header--grid">
            <div class="pinned">
                <svg><use xlink:href="#pinned--sprite"></use></svg>Pinned
            </div>
            <div class="unpin" @click="pinPost(false)">Unpin</div>
        </header>

        <!-- //////content////// -->
        <div class="new-post--content" :class="{'fullWidth' : swiper_position === 'bottom'}">
            <div :class="['new-post--media-bg',{'new-post--media-bg-color': post_spec==='motom' && post.influencer_id != post.post_author_id && post.post_type != 'IMAGE' && post.post_type != 'THISORTHAT'}]">
                <div class="new-post--media">
                    <div v-if="post_spec==='motom' && display_type=='grid'" class="img-icon">
                        <Icon :type="post.post_type.toLowerCase()" :icon="post.post_type.toLowerCase() + '-no-circle'" :size="'small'" :shape="'circle'" />
                    </div>
                    <div v-if="post.payload && !is_fan_made && is_connected && post.influencer_id === post.post_author_id" :class="['new-post--main',
                        {'space-around': display_type === 'feed' && (!post.tagged_products || post.tagged_products.length === 0)}, 
                        {'space-part': display_type === 'feed' && post.tagged_products && post.tagged_products.length !== 0}]"
                        @click="gotoPost(post.post_type,post.social_post_url)">
                        <div v-if="post.post_type === 'YOUTUBE'" class="video-play"><svg><use xlink:href="#tiktop-player-play--sprite"></use></svg></div>
                        <img :src="Get_Image(post.payload.thumbnail_url, '')" @error="imageErrorInstagram($event)" >
                    </div>
                    <div v-else-if="post.tiktok_payload && !is_fan_made && is_connected && post.influencer_id === post.post_author_id" :class="['new-post--main',
                        {'space-around': display_type === 'feed' && (!post.tagged_products || post.tagged_products.length === 0)}, 
                        {'space-part': display_type === 'feed' && post.tagged_products && post.tagged_products.length !== 0}]"
                        @click="gotoPost(post.post_type,post.social_post_url)">
                        <div v-if="post.post_type === 'TIKTOK'" class="video-play"><svg><use xlink:href="#tiktop-player-play--sprite"></use></svg></div>
                        <img :src="Get_Image(post.tiktok_payload.thumbnail_url, '')" @error="imageErrorTiktok($event)" >
                    </div>
                    <div v-else class="new-post--main" @click="gotoPostDetail(post.social_post_url)"
                    :class="{'space-around':  (!post.tagged_products || post.tagged_products.length === 0) && (post.post_type === 'IMAGE' || post.post_type === 'THISORTHAT' || post.post_type === 'TIKTOK'),
                            'space-part':  post.tagged_products && post.tagged_products.length !== 0 && (post.post_type === 'IMAGE' || post.post_type === 'THISORTHAT' || post.post_type === 'TIKTOK')}">
                        <section v-if="post.post_type === 'INSTAGRAM'" class="new-post--instagram-embed" :data-link="post.social_post_url">
                            <blockquote class="instagram-media" :data-instgrm-permalink="post.social_post_url" data-instgrm-version="12"></blockquote>
                            <div class="bg" style="padding-bottom: 100%;"></div>
                        </section>
                        <section v-else-if="post.post_type === 'IMAGE'" class="new-post--image-item">
                            <div v-for="(image, index) in post.image_list" :key="index">
                                <img v-if="image.is_preview === true" :src="image.image_list_src" >
                                <img v-else-if="image.url" :src="Get_Image(image.url + image.dir + '400x400/' + image.file_name)" >
                                <img v-else :src="Get_Image(image.dir + '400x400/' + image.file_name)" >
                            </div>
                        </section>
                        <section v-else-if="post.post_type === 'THISORTHAT'" class="new-post--tot-item">

                            <div class="tot-main-container--wrapper" v-if="post.left_image && post.right_image && post.left_image.data && post.right_image.data">
                                <div class="tot-ico">OR</div>
                                <div class="tot-main-container--tot-left" @click="totVote(post.post_id ,'left')" :class="{'hasVotes': post.tapped_state.tapped && (post.tapped_percentage.left > 0 || post.tapped_percentage.right > 0), 'win': post.tapped_state.tapped && (post.tapped_percentage.left > post.tapped_percentage.right)}">
                                    <div class="tot-container">
                                        <canvas width="100" height="100" class="tot-container--img-outer"></canvas>
                                        <div class="tot-container--img-center">
                                            <canvas :width="post.left_image.data.left_width" :height="post.left_image.data.left_height" class="tot-container--img-bg"></canvas>
                                            <div class="tot-container--img-scale" :style="{
                                                transform: ' scale('+post.left_image.data.left_scale+') !important'
                                            }">
                                                <div class="tot-container--img-translate" :style="{
                                                    transform: 'translate('+post.left_image.data.left_translate_left_position+'%, '+post.left_image.data.left_translate_top_position+'%) !important'
                                                }">
                                                    <img v-if="post.left_image.is_preview" :src="post.left_image.file_name" class="tot-container--img" />
                                                    <img v-else :src="Get_Image(post.left_image.url + post.left_image.dir  + '400x400/' +  post.left_image.file_name)" class="tot-container--img">
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="tot-voteResult" :class="{'hasVotesWin': post.tapped_percentage.left > post.tapped_percentage.right}" v-if="post.tapped_state.tapped && (post.tapped_percentage.left > 0 || post.tapped_percentage.right > 0)" v-html="showVotePercentage('left')"></div>
                                </div>

                                <div class="tot-main-container--tot-right" @click="totVote(post.post_id ,'right')" :class="{'hasVotes': post.tapped_state.tapped && (post.tapped_percentage.left > 0 || post.tapped_percentage.right > 0), 'win': post.tapped_state.tapped && (post.tapped_percentage.right > post.tapped_percentage.left)}">
                                    <div class="tot-container">
                                        <canvas width="100" height="100" class="tot-container--img-outer"></canvas>
                                        <div class="tot-container--img-center">
                                            <canvas :width="post.right_image.data.right_width" :height="post.right_image.data.right_height" class="tot-container--img-bg"></canvas>
                                            <div class="tot-container--img-scale" :style="{
                                                transform: ' scale('+post.right_image.data.right_scale+') !important'
                                            }">
                                                <div class="tot-container--img-translate" :style="{
                                                    transform: 'translate('+post.right_image.data.right_translate_left_position+'%, '+post.right_image.data.right_translate_top_position+'%) !important'
                                                }">
                                                    <img v-if="post.right_image.is_preview" :src="post.right_image.file_name" class="tot-container--img" />
                                                    <img v-else :src="Get_Image(post.right_image.url + post.right_image.dir + '400x400/' + post.right_image.file_name)" class="tot-container--img">
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="tot-voteResult" :class="{'hasVotesWin': post.tapped_percentage.right > post.tapped_percentage.left}" v-if="post.tapped_state.tapped && (post.tapped_percentage.left > 0 || post.tapped_percentage.right > 0)" v-html="showVotePercentage('right')"></div>
                                </div>
                            </div>
                        </section>
                        <section v-else-if="post.post_type === 'YOUTUBE' && post.social_post_url" class="new-post--video-item">
                            <div class="embed-video">
                                <youtube :video-id="getIdFromURL(post.social_post_url)" :player-vars="{ playsinline: 1 }" @error="videoError" @ready="videoReady"></youtube>
                            </div>
                        </section>
                        <section v-else-if="post.post_type === 'TIKTOK' && is_post_details" class="new-post--tiktok-item">
                            <div class="embed-tiktok" v-html="embed_data" :class="{'is-iphone': is_iphone}">
                            </div>
                            <script type="application/javascript" src="//www.tiktok.com/embed.js"/>
                        </section>
                        <section v-else-if="post.post_type === 'TIKTOK'" class="new-post--tiktok-item">
                            <TiktokEmbed :post="post" />
                        </section>
                    </div>
                </div>
                <aside class="new-post--tagged-products" v-if="swiper_position === 'right'" :class="{'space': display_type === 'feed' && (post.tiktok_payload || post.payload)}">
                    <div class="label">
                        <div class="tagged-ico"><svg><use xlink:href="#bag-outline--sprite"></use></svg></div><span class="text">Shop</span>
                    </div>
                    <div class="slider-wrap">
                        <div class="product-list-wrapper">
                            <ul class="product-list">
                                <li v-for="(product, index) in post.tagged_products" :key="product.id +'-'+ index">
                                    <Product :show_info_2="true" :hide_like_button="true" :product="product"/>
                                </li>
                            </ul>
                        </div>
                        <Button :type="'bg'" @click.native="showTaggedPopup(false, true)" :color="'orange'" :size="'20'">See All</Button>
                    </div>
                </aside>
                <div class="new-post--tagged-products-bottom" v-if="swiper_position === 'bottom'">
                    <div class="label">
                        <div class="tagged-ico">
                            <svg><use xlink:href="#bag-outline--sprite"></use></svg>
                        </div>
                        <div class="text" v-if="$route.meta === 'private-profile'">Shop my post</div>
                        <div class="text" v-else>Shop this post</div>
                        <div class="button-wrapper">
                            <Button :type="'bg'" @click.native="showTaggedPopup(false, true)" :color="'orange'" :size="'20'">See All</Button>
                        </div>
                    </div>
                    <swiper class="swiper" v-if="post.tagged_products" :options="bottom_product_swiper" ref="bottomSwiper">
                        <swiper-slide v-for="(product, index) in post.tagged_products" :key="product.id +'-'+ index">
                            <Product :show_info_2="true" :hide_like_button="true" :product="product"/>
                        </swiper-slide>
                        <div class="arrow prev" slot="button-prev">
                            <svg><use xlink:href="#arrow--sprite"></use></svg>
                        </div>
                        <div class="arrow next" slot="button-next">
                            <svg><use xlink:href="#arrow--sprite"></use></svg>
                        </div>
                    </swiper>
                </div>
            </div>
        </div>
        
        <!-- //////footer////// -->
        <footer v-if="display_type=='grid'" class="new-post--footer--grid">
            <div class="is-link" v-if="post_spec==='social'" :class="[post.post_type.toLowerCase()]" @click="openLink">
                <Icon :type="post.post_type.toLowerCase()" :icon="post.post_type.toLowerCase() + '-no-circle'" :size="'small'" :shape="'circle'" />
                <div v-if="post.social_username" class="user-id"><span v-if="post.post_type != 'YOUTUBE' && post.post_type !== 'TIKTOK'">@</span>{{post.social_username}}</div>
            </div>
            <div v-else>
                <Avatar v-if="post.post_author_id != this.$store.state.user.id" :to="`/user/${post.post_author_id}/profile`" :image="post.post_author_image" :type="post.post_author_type" :size="'extra-small'" :is_influencer="Boolean(post.is_influencer)"/>
                <Avatar v-else :to="this.$route.name === 'me' ? '#' : '/me'" :image="post.post_author_image" :type="post.post_author_type" :size="'extra-small'" :is_influencer="Boolean(post.is_influencer)"/>
            </div>
            <div class="icon-list">
                <ul>
                    <li v-if="post_spec==='motom'" :class="['ico','js-post-likeid-' + post.post_id, {'is-liked': post.is_liked}]"><div ref="like" @click="like()"><svg><use xlink:href="#like2--sprite"></use></svg></div></li>
                    
                    <li 
                    @click="tagNewProducts" 
                    :class="{'is-tagged': checkTaggedBySelf && $route.meta === 'private-profile'}" 
                    >
                        <div class="ico">
                            <svg v-if="$route.name == 'public-profile-social' && post.tagged_products && post.tagged_products.length !== 0"><use xlink:href="#bag-fill--sprite"></use></svg>
                            <svg v-else-if="$route.name == 'public-profile-social'"><use xlink:href="#bag-outline--sprite"></use></svg>
                            <svg v-else-if="post.tagged_products && post.tagged_products.length !== 0"><use xlink:href="#tag-offer-fill--sprite"></use></svg>
                            <svg v-else><use xlink:href="#tag-offer-outline--sprite"></use></svg>
                        </div>
                    </li>
                    <li v-if="$store.state.is_logged"><div @click="openEditMenu" class="ico"><svg><use xlink:href="#post-pop-up-menu--sprite"></use></svg></div></li>
                </ul>
            </div>
        </footer>
        <footer v-else>
            <div class="new-post--footer--feed">
                <ul>
                    <li v-if="post_spec==='motom'" :class="['js-post-likeid-' + post.post_id, {'is-liked': post.is_liked}]" ref="like" @click="like()"><div class="ico"><i><svg><use xlink:href="#like2--sprite"></use></svg></i> <span>Like</span></div></li>
                    <li v-if="post.tagged_products && post.tagged_products.length !== 0" @click="share()"><div class="ico"><i><svg><use xlink:href="#share--sprite"></use></svg></i> <span>Share</span></div></li>
                    <li v-else-if="post.post_type !== 'INSTAGRAM' && post.post_type !== 'TIKTOK'" @click="share()"><div class="ico"><i><svg><use xlink:href="#share--sprite"></use></svg></i> <span>Share</span></div></li>
                    <li v-else-if="$route.name === 'public-profile' || $route.name === 'public-profile-social' || $route.name === 'private-profile-me' || $route.name === 'private-profile-social'" @click="share()"><div class="ico"><i><svg><use xlink:href="#share--sprite"></use></svg></i> <span>Share</span></div></li>
                    <li 
                    @click="tagNewProducts" 
                    :class="{'is-tagged': checkTaggedBySelf && $route.meta === 'private-profile'}" 
                    >
                        <div class="ico">
                            <i>
                                <svg v-if="post.tagged_products && post.tagged_products.length !== 0"><use xlink:href="#tag-offer-fill--sprite"></use></svg>
                                <svg v-else><use xlink:href="#tag-offer-outline--sprite"></use></svg>
                            </i>
                            <span>Add tag</span>
                        </div>
                    </li>

                </ul>
            </div>

        </footer>
    </article>
</template>

<script>
/* eslint-disable */
import { EventBus } from '@/events/bus.js';
import { Swiper, SwiperSlide } from 'vue-awesome-swiper';
import Product from '@/components/product/SearchProductItem.vue';
import TiktokEmbed from '@/components/ui/TiktokEmbed.vue';
import Icon from '@/components/ui/Icon.vue';
import Button from '@/components/ui/Button.vue';
import Avatar from '@/components/ui/Avatar.vue';
import { getIdFromURL } from 'vue-youtube-embed';
import 'swiper/css/swiper.css';

import '@/assets/arrow-down.svg?sprite';
import '@/assets/arrow-up.svg?sprite';
import '@/assets/create.svg?sprite';
import '@/assets/arrow.svg?sprite';
import '@/assets/arrow-select.svg?sprite';
import '@/assets/share.svg?sprite';
import '@/assets/like2.svg?sprite';
import '@/assets/back.svg?sprite';
import '@/assets/post-pop-up-menu.svg?sprite';
import '@/assets/instagram-no-circle.svg?sprite';
import '@/assets/youtube-no-circle.svg?sprite';
import '@/assets/checked-circle-2.svg?sprite';
import '@/assets/pinned.svg?sprite';
import '@/assets/bag-outline.svg?sprite';
import '@/assets/bag-fill.svg?sprite';
import '@/assets/tag-offer-outline.svg?sprite';
import '@/assets/tag-offer-fill.svg?sprite';
import '@/assets/tiktop-player-play.svg?sprite';


export default {
    name: 'post',
    props: {
        post: Object,
        is_likeboard_post: Boolean,
        is_my_likeboard: Boolean,
        likeboard_id: Number,
        display_type: String,
        post_spec: String,
        is_post_details: Boolean,
        is_connected: Boolean,
        is_fan_made: Boolean,
    },

    data() {
        return {
            is_error: false,
            is_tagged: false,
            is_iphone: false,
            show_all_products: false,
            default_CMSUser_image: '',
            default_image: '',
            product_swiper: {
                direction: 'vertical',
                slidesPerView: 4,
                spaceBetween: 6,
                navigation: {
                    nextEl: '.next',
                    prevEl: '.prev'
                }
            },
            bottom_product_swiper: {
                slidesPerView: 4,
                spaceBetween: 6,
                navigation: {
                    nextEl: '.next',
                    prevEl: '.prev'
                }
            },
            swiper_position: '',
            embed_data: '',
            show_pin_modal: false,
        };
    },

    methods: {
        getIdFromURL: getIdFromURL,

        openLink() {
            if (this.post.social_username) {
                let name = this.post.social_username;
                let url = '';

                switch (this.post.post_type) {
                case 'INSTAGRAM':
                    url = 'https://www.instagram.com/' + name;
                    break;

                case 'YOUTUBE':
                    url = this.post.social_profile_url;
                    break;

                case 'TIKTOK':
                    url = 'https://www.tiktok.com/' + name;
                    break;
                }

                if (url) window.open(url, '_blank').focus();
            } 
        },

        like() {
            if (this.$store.state.is_logged) {
                const obj = {};
                obj.type = 'post';
                obj.post = this.post;
                obj.unique_key = this.post.unique_key;
                obj.is_liked = this.$refs.like.classList.contains('is-liked');
                obj.is_likeboard_post = this.is_likeboard_post;
                obj.is_my_likeboard = this.is_my_likeboard;
                obj.likeboard_id = this.likeboard_id;

                EventBus.$emit('open-add-likeboards', obj);
            } else {
                this.$store.commit('login', {
                    is_show: true,
                    title: 'Like for LikeBoards!',
                    text: 'Save this post to a LikeBoard by logging in below or hitting the Sign up button to create an account.'
                });
            }
        },

        showProductPopup(product_id) {
            const obj = {};
            obj.product_id = product_id;
            EventBus.$emit('open-product-popup', obj);
        },

        videoReady(event) {
            event.target.playVideo();
            setTimeout(() => {
                event.target.stopVideo();
            }, 100);
        },

        videoError() {
            if (!this.is_error) {
                this.is_error = true;
                console.log('error video', this.post);
                EventBus.$emit('disable-post', {
                    post_id: this.post.post_id
                });
            }
        },

        showTaggedPopup(is_empty = false, is_see_all = false) {
            const obj = {};
            let is_have_who_tagged = true;
            obj.is_empty = is_empty;
            obj.is_see_all = is_see_all
            obj.post = JSON.parse(JSON.stringify(this.post));
            obj.post.tagged_products.forEach(product => {
                if (typeof product.who_tagged === 'undefined') is_have_who_tagged = false;
            });

            if (is_have_who_tagged) {
                EventBus.$emit('open-tagged-popup', obj);
            } else {
                this.$store.state.show_loading = true;

                this.$ajax('/api/post/' + this.post.unique_key, {
                    method: 'get',
                    withCredentials: true
                }).then(res => {
                    if (res.data.status === 'success') {
                        obj.post = res.data.data;
                        EventBus.$emit('open-tagged-popup', obj);
                    }

                    this.$store.state.show_loading = false;
                });
            }
        },

        tagNewProducts() {
            if (this.display_type === 'feed') {
                this.showTaggedPopup(true);
            } else if (this.display_type === 'grid') {
                this.showTaggedPopup();
            } else {
                const obj = {};
                obj.post = this.post;
                EventBus.$emit('tag-new-products', obj);
            }
        },

        totVote(postID, leftRight) {
            const self = this;

            if (this.$store.state.is_logged === false) {
                this.$store.commit('login', {
                    is_show: true,
                    title: 'Make your vote count!',
                    text: 'You need to log in for that or click the Sign up button below to create your Motom account.'
                });
                return;
            }

            const formData = new FormData();

            const d = Date.now();

            this.$ajax('/user/thisorthat/' + postID + '/tap_' + leftRight + '?d=' + d, {
                method: 'post',
                data: formData,
                withCredentials: true
            }).then(res => {
                if (res.data.status === 'success') {
                    self.post.tapped_state.tapped = true;
                    self.post.tapped_state.selected = leftRight;

                    if (leftRight === 'left') {
                        self.post.tapped_percentage.left++;
                    } else if (leftRight === 'right') {
                        self.post.tapped_percentage.right++;
                    }
                }
            }).catch(err => {
                console.log(err);
            });
        },

        showVotePercentage(leftRight) {
            const self = this;

            const leftPercentage = ((self.post.tapped_percentage.left * 100.0) / ((self.post.tapped_percentage.left + self.post.tapped_percentage.right) * 1.0)).toFixed(0);
            const rightPercentage = 100 - leftPercentage;

            if (leftRight === 'left') {
                return leftPercentage + '%';
            } else if (leftRight === 'right') {
                return rightPercentage + '%';
            }
        },

        gotoProfile() {
            if (this.post.post_author_id !== this.$store.state.user.id) {
                const profileURL = '/user/' + this.post.post_author_id + '/profile';
                this.$router.push(profileURL);
            } else {
                if (this.$route.name !== 'me') {
                    this.$router.push('/me');
                }
            }
        },

        popupPosts() {
            EventBus.$emit('popup-posts', {
                type: 'post',
                posts: this.post.in_post
            });
        },

        createPost() {
            if (this.$store.state.is_logged) {
                let type = '';

                switch (this.post.post_type) {
                case 'INSTAGRAM':
                    type = 'instagram';
                    break;

                case 'YOUTUBE':
                    type = 'video';
                    break;

                case 'TIKTOK':
                    type = 'tiktok';
                    break;
                }

                setTimeout(() => {
                    EventBus.$emit('create-post', {
                        type: type,
                        post: this.post
                    });
                }, 500);

                this.$router.push({ path: '/create-post/' + type });
            } else {
                this.$store.commit('login', {
                    is_show: true,
                    title: 'Ready to create a post?',
                    text: 'You need to log in for that or click the Sign up button below to create your Motom account.'
                });
            }
        },

        share() {
            let label = window.user_id_alpha ? window.user_id_alpha : 'undefined';
            label = label + ',' + this.post.unique_key + ',,';
            this.GA_Tracking('SHARE', 'CLICK', label);

            const obj = {};
            obj.description = this.post.post_caption;
            obj.shareURL = `${window.location.origin}/postdetails/${this.post.unique_key}`;
            obj.postType = this.post.post_type;
            obj.share_post = this.post;
            
            EventBus.$emit('open-post-share', obj);
        },

        openEditMenu() {
            const obj = {};
            obj.post = this.post;
            EventBus.$emit('open-edit-menu', obj);
        },

        openMoreMenu() {
            this.$refs.edit_menu.open();
        },

        closeMoreMenu() {
            this.$refs.edit_menu.close();
        },

        saveUserJourneyPostInLikeBoard() {
            const self = this;
            const postID = self.post.unique_key;
            if (postID) {
                self.$store.commit('setPostID', self.post.unique_key);
            } else {
                self.$store.commit('setPostID', '');
            }
        },

        gotoPost(type,url) {
            if (type === 'YOUTUBE' || type === 'TIKTOK') {
                let obj = {};
                obj.post = this.post;
                EventBus.$emit('popup-tiktok', obj);
            }
            else
                window.open(url, '_blank');
        },        

        pinPost(is_pin) {
            let self = this;
            let url = is_pin ? '/api/pin' : '/api/unpin';
            this.$ajax(url, {
                method: 'post',
                withCredentials: true,
                data: {
                        item_id: self.post.post_id,
                        item_type: this.$route.name == 'private-profile-social' ? 0 : 1
                        }
                }).then(res => {
                    self.$set(self.post, 'is_pin', true);
                    EventBus.$emit('upPinPost', {
                        post: self.post.post_id,
                        is_pin: false,
                    });
                })
                .catch(error => {
                    console.log(error)
                })
        },
    },

    computed: {
        captionAddTag() {
            let caption = this.post.post_caption;
            caption = caption.replace(/(\B#\w*[a-zA-Z]+\w*)/gi, '<hashtag>$1</hashtag>');
            caption = caption.replace(/(\B@\w*[a-zA-Z]+\w*)/gi, '<userat>$1</userat>');
            return caption;
        },

        checkTaggedBySelf() {
            return this.post.tagged_products.filter(p => p.who_tagged.user_id === this.post.post_author_id).length > 0 ? true : false;
        },

    },

    components: {
        Product,
        Swiper,
        SwiperSlide,
        Icon,
        Button,
        TiktokEmbed,
        Avatar,
    },

    mounted () {
        const self = this;

        this.is_iphone = !window.MSStream && /iPhone|iPod/.test(navigator.userAgent);

        if (typeof self.post !== 'undefined' && typeof self.post.post_type !== 'undefined' && self.post.post_type === 'TIKTOK') {
            if (self.post.tiktok_payload) {
                var n = self.post.tiktok_payload.html.indexOf('<script');
                self.embed_data = self.post.tiktok_payload.html.substr(0, n);
            } else {
                self.embed_data = self.post.embed_data;
            }
        }

        if (typeof self.post !== 'undefined' && typeof self.post.post_type !== 'undefined' && self.post.post_type === 'INSTAGRAM') {
            setTimeout(() => {
                if (window.instgrm && window.instgrm.Embeds) window.instgrm.Embeds.process();
            }, 500);
        }


        if ((this.display_type === 'feed' && this.post.tagged_products && this.post.tagged_products.length !== 0 && (this.post.post_type === 'IMAGE' || this.post.post_type === 'THISORTHAT' || this.post.post_type === 'TIKTOK' ) && this.is_likeboard_post !== true) || (this.$route.meta === 'private-profile' && this.display_type === 'feed' && this.post.tagged_products && this.post.tagged_products.length !== 0 && this.is_likeboard_post !== true)) {
            this.swiper_position = 'bottom';
        } else if (this.display_type ==='feed' && (this.post.tagged_products && this.post.tagged_products.length !== 0 && (this.post.post_type === 'INSTAGRAM' || this.post.post_type === 'YOUTUBE' )) && this.is_likeboard_post !== true) {
            this.swiper_position = 'right';
        }

        self.$nextTick(() => {
            self.default_CMSUser_image = require('@/assets/images/Motom-Creator.png');

            self.default_image = require('@/assets/images/default-user.png');

            if (self.post.tagged_products && self.post.tagged_products.length !== 0) {
                self.post.tagged_products.forEach(product => {
                    if (typeof product.who_tagged !== 'undefined' && product.who_tagged.is_me) self.is_tagged = true;
                    product.brand_id = 0;
                    product.brand_name = '';
                    product.brand_logo = '';

                    if (product.brand && product.brand.parent_id && product.brand.parent) {
                        product.brand_id = product.brand.parent_id;

                        if (product.brand.parent.name) {
                            product.brand_name = product.brand.parent.name;
                        } else {
                            product.brand_name = product.brand.parent.title;
                        }

                        product.brand_logo = product.brand.parent.image;
                    } else if (product.brand && !product.brand.parent_id) {
                        product.brand_id = product.brand.id;

                        if (product.brand.name) {
                            product.brand_name = product.brand.name;
                        } else {
                            product.brand_name = product.brand.title;
                        }

                        product.brand_logo = product.brand.image;
                    }
                });
            }
        });
    }
};
</script>

<style lang="scss">
    @import '../resources/sass/_variables.scss';
    @import '@/sass/desktop_variables.scss';

    .new-post {
        padding: .8rem;
        border-radius: 1.4rem;
        max-width: var(--post-max-width);;
        width: 100%;
        //margin: 0 auto $padding auto;
        margin: 0 auto 1.6rem auto;
        box-sizing: border-box;
        overflow: hidden;
        background-color: $white;
        position: relative;
        z-index: 1;
        box-shadow: 0 2px 0 rgba(0, 0, 0, .1);
        border: .08rem solid #f2f2f2;

        &--header--grid {
            display: flex;
            .pinned {
                // padding: 0.8rem;
                height: 28px;
                display: flex;
                align-items: center;
                color: #F15A29;
                font: var(--span);
                font-weight: var(--semibold);
                svg {
                    width: 1.1rem;
                    height: 1.1rem;
                    margin-left: 0.9rem;
                    margin-right: 0.5rem;
                }
            }
            .unpin {
                width: 100%;
                display: flex;
                justify-content: flex-end;
                align-items: center;
                color: var(--black-50);
                font: var(--span);
                font-weight: var(--semibold);
                padding: 0.8rem 0.6rem;
                cursor: pointer;
            }
        }

        &--header--feed {
            // display: flex;
            // padding: 0.8rem;
            align-items: center;
            .left {
                width: 70%;
                display: flex;
                align-items: center;
                cursor: pointer;

                .user-id {
                    margin-left: 0.8rem;
                    font: var(--p-12);
                    font-weight: var(--semibold);
                    color: var(--black-90);
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                    width: 17rem;
                }

                .ui-icon {
                    display: block;
                }
            }
            .right {
                width: 30%;
                display: flex;
                align-items: center;
                justify-content: flex-end;
                .pinned {
                    height: 28px;
                    display: flex;
                    align-items: center;
                    color: #F15A29;
                    font: var(--p-12);
                    font-weight: var(--semibold);
                    background-color: var(--bg-07);
                    border-radius: 100px;
                    padding: 0.5rem 1rem;
                    margin-right: 0.8rem;
                    svg {
                        width: 1.4rem;
                        height: 1.4rem;
                        margin-right: 0.5rem;
                    }
                }

                .edit {
                    // width: 2.4rem;
                    // padding: 0.8rem;
                    svg {
                        width: 1.6rem;
                        height: 1.6rem;
                        fill: var(--black-50);
                    }
                }
            }
        }

        &--main {
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 8px;
            img {
                border-radius: 8px;
                cursor: pointer;
            }

            .video-play {
                position: absolute;
                width: 3.2rem;
                height: 3.2rem;
                background-color: #fff;
                border-radius: 100px;
                border: 1px solid #F2F2F2;
                padding: 0.4rem;
                display: flex;
                justify-content: center;
                align-items: center;
                cursor: pointer;

                svg {
                    display: flex;
                    overflow: hidden;
                    fill: var(--black-75);
                    width: 1.7rem;
                    height: 1.7rem;
                    padding-left: 0.3rem;
                }
            }
        }

        &--footer--grid {
            padding: 0.8rem 0.9rem 0.8rem 0.6rem;
            display: flex;

            .is-link {
                cursor: pointer;
                flex: 1;
                padding-left: .2rem;
            }

            .icon-list {
                // min-width: 16rem;
                justify-content: flex-end;

                ul {
                    display: flex;
                    li {
                        display:flex;
                        align-items: center;
                        justify-content: center;
                        margin-left: .8rem;
                        cursor: pointer;

                        &:last-child {
                            width: 2.4rem;
                        }

                        &.is-tagged  {
                            svg {
                                fill: $orange;
                            }
                        }

                        span {
                            margin-left: 0.5rem;
                            font: var(--span);
                            font-weight: var(--semibold);
                        }
                        
                        svg {
                            width: 1.4rem;
                            height: 1.4rem;
                            fill: var(--black-50);
                        }
                    }
                }
            }
            
            .ui-avatar {
                $size: 2.4rem;
                min-width: $size;
                width: $size;
                height: $size;
                margin-right: unset;

                & > div {
                    // width: $size + 0.4rem;
                    border: none;
                }

                img {
                    width: $size;
                    height: $size;
                }
            }

            .ui-icon {
                height: 1.8rem !important;
                width: 1.8rem !important;
                min-height: 1.8rem;
                min-width: 1.8rem;

                svg {
                    height: 1rem !important;
                    width: 1rem !important;
                }
            }

            div {
                display:flex;
                align-items: center;
            }

            .user-id {
                display: block;
                justify-content: flex-start;
                margin-left: 0.8rem;
                font: var(--p-12);
                font-weight: var(--semibold);
                color: var(--black-90);
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
                width: 15rem;

                @media (max-width: $mobile_size) {
                    & {
                        width: 7rem;
                    }
                }
            }
        }

        &--footer--feed {
            padding: 0.8rem;

            ul {
                display: flex;
                justify-content: center;

                li {
                    width: 100%;
                    height: 3.2rem;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    position: relative;

                    .ico {
                        display: flex;
                        align-items: center;
                        cursor: pointer;
                        color: $med_grey;
                        font-weight: 600;
                        font-size: 1.2rem;

                        i {
                            display: block;
                            height: 2rem;
                            width: 2rem;
                            margin-right: .5rem;
                        }

                        svg {
                            height: 100%;
                            width: 100%;
                            fill: $med_grey;

                            &.active {
                                fill: $orange;
                            }
                        }
                    }
                    
                    &.is-tagged .ico {
                        color: $orange;

                        svg {
                            fill: $orange;
                        }
                    }

                    &.is-liked .ico {
                        color: $orange;

                        svg {
                            fill: $orange;
                        }
                    }
                }
            }
        }

        img {
            width: 100%;
            height: 100%;
            object-fit:cover;
            // border-radius: 8px;
        }

        .tot-container .img {
            position: absolute;
            left: 0;
            top: 0;
            height: 100%;
            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            img {
                display: block;
                max-width: 100%;
                max-height: 100%;
            }
        }

        a {
            text-decoration: none;
        }

        ul, li {
            padding: 0;
            margin: 0;
            list-style: none;
        }

        // &--caption {
        //     @include medium-14($dark_grey);
        //     letter-spacing:  -.01rem;
        //     padding-top: 0.2rem;
        //     //padding-left: 1rem;
        //     display:block
        // }

        hashtag {
            color: $orange;
        }

        userat {
            color: $instagram;
        }

        &--author-info {
            // margin: 1.2rem 1.2rem 0 1.2rem;
            display: flex;
            align-items: center;
            @include regular-12;
            border-bottom: 0.1rem solid #F2f2f2;
            margin: unset;
            border-bottom: none;
            letter-spacing: unset;

            &.social {
                padding: 0.8rem;
            }
            &.motom {
                padding: 1.2rem 1.2rem 1.6rem 1.2rem;
                flex-wrap: wrap;

                .pinned {
                    height: 28px;
                    display: flex;
                    align-items: center;
                    color: #F15A29;
                    font: var(--p-12);
                    font-weight: var(--semibold);
                    // background-color: var(--bg-07);
                    // border-radius: 100px;
                    padding: 0.5rem 1rem;
                    margin-right: 0.8rem;
                    svg {
                        width: 1.4rem;
                        height: 1.4rem;
                        margin-right: 0.5rem;
                    }
                }

                .caption {
                    @include medium-14($dark_grey);
                    width: 100%;
                    letter-spacing:  -.01rem;
                    padding-top: 1rem;
                    display:block
                }
            }

            .avatar {
                $size: 4.6rem;
                min-width: $size;
                width: $size;
                height: $size;
                display: block;
                margin-bottom: auto;

                & > div {
                    display: block;
                    width: $size;
                    height: $size;
                    border-radius: 50%;
                    position: relative;
                    border: solid 0.06rem #E0E0E0;
                    overflow: hidden;
                }

                img {
                    width: $size;
                    height: $size;
                    object-fit: cover;
                    left: 0;
                    top: 0;
                }
            }

            .post-avatar {
                $size: 4rem;
                min-width: $size;
                width: $size;
                height: $size;
                display: block;
                margin-bottom: auto;

                border-radius: 50%;
                display:flex;
                align-items: center;
                justify-content: center;
                align-self:left;
                position: relative;
             }

            .info {
                flex: 1;
                padding: 0 1rem;
                overflow: hidden;
                white-space: nowrap;
                text-overflow: ellipsis;

                &--post-author {
                    font-weight: 600;
                    font-size: 1.4rem;
                    line-height: 1.8rem;
                    letter-spacing: -.01rem;
                    color:$dark_grey;

                }
            }
        }

        &--tag-products svg {
            height: 2rem;
            width: 2rem;
            fill: $orange;
        }

        &--edit-post {
            width: 1rem;
            display: flex;
            cursor: pointer;

            svg {
                height: 1.6rem;
                width: 1.6rem;
                fill: var(--black-50);
            }
        }

        &--content {
            width: 100%;

            .img-icon {
                position: absolute;
                margin-top: 0.6rem;
                margin-left: 0.6rem;
                .ui-icon {
                    height: 2rem !important;
                    width: 2rem !important;

                    svg {
                        height: 1.2rem !important;
                        width: 1.2rem !important;
                    }
                }
            }

            &.fullWidth > .new-post--media-bg {
                display: block;
            }

            .new-post--main {
                &.space-around {
                    padding: 0.8rem 0.8rem 0 0.8rem;
                }
                // &.space-part {
                //     padding: 0.8rem 0 0 0.8rem;
                // }
            }
        }

        &--media {
            flex: 1;
            order: 1;
            width: 100%;
            &-bg {
                display: flex;
                width: 100%;
                height: 100%;
                &-color {
                    background: linear-gradient(103.37deg, #CCDCF3 24.6%, #D3E0F4 45.09%, #DBE9FD 63.87%, #D3E0F4 86.49%, #C6D5EC 99.18%);
                    padding: 0.6rem;
                    border-radius: 0.8rem 0.8rem 0 0;
                }
            }
        }

        &--image-list {
            position: relative;

            .swiper-container {
                min-width: 0;
                width: 100%;
            }

            .swiper-slide {
                position: relative;
                width: 100%;
            }

            img {
                display: block;
                width: 100%;
            }
        }

        &--tagged-products + &--media {
            width: calc(100% - 8rem);
        }

        &--instagram-embed {
            width: 100%;
            overflow: hidden;
            position: relative;
            background-color: $grey_white;

            .bg {
                display: block;
                padding: 0 0 120%;
            }

            .instagram-media {
                width: 100% !important;
                height: 100% !important;
                min-width: unset !important;
                overflow: hidden;
                position: absolute;
                left: 0;
                top: 0;
                // border-radius: 0!important;
                box-shadow: none!important;
                display: block;
                margin: 0px !important;
                padding: 0px !important;
            }
        }

        &--image-item {
            width: 100%;
            overflow: hidden;
            position: relative;
            background-color: $grey_white;
            border-radius: 0.8rem;
            border: 0.1rem solid #f2f2f2;

            img {
                width: 100%;
                display: block;
                height: auto;
            }
        }

        &--video-item {
            flex: 1;

            .embed-video {
                width: 100%;
                overflow: hidden;
                position: relative;

                &::before {
                    display: block;
                    padding: 0 0 56.6%;
                    content: '';
                }

                iframe {
                    position: absolute;
                    left: 0;
                    top: 0;
                    height: 100% !important;
                    width: 100% !important;
                }
            }
            iframe {
                height: 100% !important;
                width: 100% !important;
                border-radius: 8px;
            }
        }

        &--tiktok-item {
            flex: 1;

            .embed-tiktok {
                width: 100%;
                overflow: hidden;
                position: relative;

                &.is-iphone iframe {
                    min-height: 774px;
                }
            }
        }

        &--tagged-products {
            $width: 8.6rem;
            min-width: $width;
            width: $width;
            padding: 0 0 0 0rem;
            padding-top: 0;
            position: relative;
            order:3;
            background-color: white;

            // &.space {
            //     margin-top: 0.8rem;
            // }

            & > .label {
                font-size: 1.2rem;
                line-height: 1.2rem;
                color:$dark_grey;
                font-weight: 700;
                background: linear-gradient(348.82deg, #C4302B 8.25%, #F0CB35 91.52%);
                -webkit-background-clip: text;
                -webkit-text-fill-color: transparent;
                margin: 0;
                text-align: center;
                display: flex;
                flex-direction: row;
                width: 8.6rem;
                align-items: center;

                .tagged-ico {
                    width: 2.4rem;
                    height: 2.0rem;
                    border-radius: 0 30px 30px 0 ;
                    background: linear-gradient(348.82deg, #C4302B 8.25%, #F0CB35 91.52%);
                    margin-right: 0.6rem;
                    display: flex;
                    justify-content: center;
                    align-items: center;

                    svg {
                        height: 1.4rem;
                        width: 1.4rem;
                        position: absolute;
                        fill: white;
                    }
                }
            }

            .slider-wrap {
                position: absolute;
                width: $width;
                left: 0;
                top: 2.5rem;
                bottom: 0;
                padding: 0.4rem;
                padding-top: 0;
                padding-bottom: 0;
                display: flex;
                flex-direction: column;

            }

            .product-list-wrapper {
                flex: 1 0 auto;
                overflow-y: auto;
                background: $grey_white;
                padding: 0.4rem;
            }

            .product-list {
                padding-top: 0;
                height: 0;

                li {
                    margin: 0 0 0.6rem 0;
                }

                &--see-all {
                    @include semi-bold-10($orange);
                    margin: auto;
                    height: 1.8rem;
                    line-height: 1.8rem;
                    background: rgba(255, 255, 255, 0.9);
                    cursor: pointer;
                }
            }
        }

        &--seemore {
            width: 100%;
            margin: .5rem 0 0;
            padding: 0.2rem 0;
            border-top: 1px solid $med_light_grey;
            text-align: right;

            a {
                @include regular-12($orange);
                position: relative;
                padding-right: 2.5rem;
            }

            svg {
                position: absolute;
                transform: rotate(180deg);
                top: .4rem;
                right: 1rem;
                height: 1rem;
                width: 1rem;
                fill: $orange;
            }
        }

        &--tagged-products-bottom {
            order: 2;
            padding-top: 0.5rem;
            background-color: white;

            .swiper-container {
                background: $grey_white;
                border-radius: 0px 0px 8px 8px;
                //padding: 0.4rem 0;
                // .product-item {

                // }
            }

            // .product-item {
            //         background:white;
            //     }
            // }
            & > .label {
                margin: 0;
                display: flex;
                align-items: center;
                padding-right: 1rem;
                height: 3.6rem;
                align-items: center;
                width: 100%;

                .tagged-ico {
                    width: 2.8rem;
                    height: 2.8rem;
                    border-radius: 0 1.4rem 1.4rem 0;
                    background: linear-gradient(348.82deg, #C4302B 8.25%, #F0CB35 91.52%);
                    margin-right: 0.6rem;
                    position: relative;

                    svg {
                        height: 1.4rem;
                        width: 1.4rem;
                        position: absolute;
                        fill: white;
                        left: 50%;
                        top: 50%;
                        transform: translate(-55%, -55%);
                    }
                }

                .text {
                    color: var(--black-75);
                    flex: 1;
                    font-weight: 600;
                }

                .button-wrapper {
                    width: 6rem;
                }
            }

            .arrow {
                height: 2.4rem;
                width: 2.4rem;
                position: absolute;
                z-index: 9;
                top: 30%;

                svg {
                    fill: #333;
                    color: #fff;
                    height: 100%;
                    width: 100%;
                }

                &.swiper-button-disabled {
                    display: none;
                }
            }

            .next {
                right: 0;
            }

            .prev {
                left: 0;

                svg {
                    transform: rotate(-180deg);
                }
            }

        }


        &--tot-item {
            width: 100%;
            overflow: hidden;
            background: $grey_white;
            border-radius: $radius;
            position: relative;
            margin: 0 0 .2rem;
            border-radius: 0.8rem;
            border: 0.1rem solid #f2f2f2;

            /*&::before {
                content: '';
                padding: 0 0 110%;
                display: block;
            }*/

            .tot-handle {
                height: 4rem;
                width: 4rem;
                line-height: 4rem;

                position: absolute;
                left: 50%;
                top: 50%;
                background-color: $orange;
                border-radius: 50%;

                margin: -2rem 0 0 -2rem;

                text-align: center;
                z-index: 9;
                color: #fff;
                font-weight: 600;
                font-size: 1.8rem;
                letter-spacing: -.1rem;
            }

            .tot-image {
                height: 100%;
                width: 50%;
                overflow: hidden;
                position: absolute;
                left: 0;
                top: 0;
                background-position: center;
                background-size: cover;
                background-repeat: no-repeat;

            }

            .tot-that {
                background-color: $light_grey;
                left: 50%;
            }

        }

        &--show-all-products {
            margin: 1rem -1rem -1rem -1rem;

            .btn {
                background: $med_dk_grey;
                position: relative;
                z-index: 999;

                span {
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    cursor: pointer;
                    text-align: center;
                    padding: .6rem;
                    font-size: 1.2rem;
                    font-weight: 500;
                    color: white;

                    svg {
                        height: 1.2rem;
                        width: 1.2rem;
                        fill: white;
                        margin-right: 1rem;
                    }

                    &.close {
                        background: $orange;
                    }
                }
            }

            .product-list {
                position: absolute;
                z-index: 998;
                background: white;
                bottom: 0;
                padding: 1rem 1rem 3rem;
                box-sizing: border-box;
                width: 100%;
                max-height: 100%;
                display: grid;
                grid-template-rows: auto 1fr;
                grid-template-columns: 100%;
                transform: translate(0, 100%);
                transition: transform .3s ease-in-out;
                border-radius: 1.4rem 1.4rem 0 0;

                & > h2 {
                    display: flex;
                    align-items: center;
                    color: $orange;
                    width: 100%;
                    padding-bottom: 1rem;
                    border-bottom: 1px solid $light_grey;
                    font-weight: 500;

                    svg {
                        height: 2rem;
                        width: 2rem;
                        margin-right: 1rem;
                    }
                }

                .list {
                    margin: 1rem 0;
                    display: flex;
                    flex-wrap: wrap;
                    overflow: auto;
                    padding: .5rem;
                    background: $grey_white;
                    border-radius: .8rem;

                    .product-item {
                        width: 33.33%;
                        padding: .5rem;
                        background:white;
                        span {
                            box-shadow: none;
                        }
                    }
                }
            }

            .bg {
                position: absolute;
                z-index: 991;
                background: rgba(0, 0, 0, 0.5);
                bottom: 0;
                left: 0;
                width: 100%;
                height: 100%;
                opacity: 0;
                pointer-events: none;
                transition: opacity .3s ease-in-out;
            }

            &.show {
                .product-list {
                    transform: translate(0, 0%);
                }

                .bg {
                    opacity: 1;
                    pointer-events: unset;
                }

            }
        }

        &--live-link {
            background: var(--cl-primary) linear-gradient(var(--mbg-bttf));
            border-radius: 1rem;
            margin: .8rem 0;

            a {
                display: flex;
                align-items: center;
                color: var(--bg-01);
                padding: .8rem;
            }

            p {
                padding-right: .8rem;
                font-size: 1.2rem;
                font-weight: 600;
            }

            .ui-button.color-white {
                max-width: 8rem;
                color: var(--black-75);
                font-size: 1rem;
                font-weight: 600;
            }
        }
    }

    .popup-posts--body .post {
        padding: 1rem;
    }

    .new-post.oc-post,
    .new-post.scraped-post {
        .to-view-posts,
        .to-create {
            display: flex;
            align-items: center;
            // padding: unset;
            font-weight: 600;
            font-size: 1.2rem;
            cursor: pointer;
            background: #fafafa;
            border-bottom: 1px solid rgba(130, 130, 130, 0.1);
            position: relative;
            margin: unset;
            // margin-bottom: 1rem;

            i {
                height: 1.6rem;
                width: 1.6rem;
                position: absolute;
                left: 1rem;
                top: .9rem;

                svg {
                    height: 100%;
                    width: 100%;
                    fill: #0f9d58;
                }
            }

            p {
                flex: 1;
                padding: 0 2rem 0 2.4rem;
                color: #0f9d58;
            }

            span {
                color: $orange;
                position: relative;
                padding-right: 1.4rem;

                svg {
                    fill: $orange;
                    height: 1rem;
                    width: 1rem;
                    position: absolute;
                    top: 50%;
                    right: 0;
                    transform: translate( 0, -40%) rotate(-90deg);
                }
            }
        }

        .to-view-posts {
            i {

                svg {
                    fill: $green;
                }
            }

            p {
                color: $green;
            }
        }

        .new-post--author-info {
            // margin: unset; //margin: 0 0 1rem;
            // border-bottom: none;
            // padding-bottom: 0;

            .avatar {
                $size: 2.4rem;
                min-width: $size;
                width: $size;
                height: $size;

                & > div {
                    width: $size;
                    height: $size;
                }
            }

            .post-avatar {
                $size: 2.4rem;
                min-width: $size;
                width: $size;
                height: $size;
                border-radius: unset;
             }
        }

        .new-post--edit-post {
            height: 1.6rem;
            position: relative;
            top: -.2rem;

            svg {
                height: 1.6rem;
            }
        }

        // .new-post--footer {
        //     margin: 0;
        //     border-top: none;
        // }
    }

    .new-post--instagram-embed {
        border-radius: .8rem;
        position: relative;

        &::before {
            position: absolute;
            left: 0;
            top: 0;
            height: 99.5%;
            width: 99.5%;
            content: '';
            z-index: 999;
            border-radius: .8rem;
            border: 1px solid $grey_white;
            pointer-events: none;
        }
    }
</style>
