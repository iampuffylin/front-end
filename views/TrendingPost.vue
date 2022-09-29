<template>
    <div class="container">
        <TrendingProfile></TrendingProfile>
        <div class="content">
            <div class="list-3" v-if="$store.state.type === 'mobile'">
                <!-- <Loading v-if="brands.length === 0" :classname="['static']" /> -->
                <div class="list-header"><div class="header-black">Featured Creators</div></div>
                <Swiper :options="creators_swiper">
                    <SwiperSlide v-for="(item, index) in featured_creators" :key="'featured-creators-'+index">
                        <div class="brand-logo" @click="viewUser(item.user_id)">
                            <Avatar :size="'large'" :image="item.avatar" />
                        </div>
                        <div class="brand-name"><span>{{ item.nickname }}</span></div>
                        <div class="brand-btn"><Button :type="'bg'" :size="'32'" :color="'orange'" @click.native="follow(item, index)" :class="{'disabled':item.is_followed}">{{ item.is_followed ? 'Following' : 'Follow' }}</Button></div>
                    </SwiperSlide>
                </Swiper>
            </div>
            <div class="right">
                <div class="list-4">
                    <!-- <Loading v-if="brands.length === 0" :classname="['static']" /> -->
                    <Post v-for="(post, index) in related_posts" :key="index" v-bind:post="post" />
                </div>

                <div class="list-3" v-if="$store.state.type != 'mobile'">
                    <!-- <Loading v-if="brands.length === 0" :classname="['static']" /> -->
                    <div class="list-header"><div class="header-black">Featured Creators</div></div>
                    <div class="list">
                        <div class="card" v-for="(item, index) in featured_creators" :key="'featured-creators-'+index">
                            <div class="brand-logo" @click="viewUser(item.user_id)">
                                <Avatar :size="'large'" :image="item.avatar" />
                            </div>
                            <div class="brand-name"><span>{{ item.nickname }}</span></div>
                            <div class="brand-btn"><Button :type="'bg'" :size="'32'" :color="'orange'" @click.native="follow(item, index)" :class="{'disabled':item.is_followed}">{{ item.is_followed ? 'Following' : 'Follow' }}</Button></div>
                        </div>
                    </div>
                    
                </div>
            </div>
        </div>

        <Loading v-if="is_loading_posts" :classname="['static']" :requestURL="requestURL"/>
        <Share :title="'Check out this Motom Trending!'" ref="share" @copied="shareCallback($event)" />

        <div class="share-button" @click="share" :class="{'is-sticky': is_sticky}">
            <Button :size="'40'" :type="'bg'" :color="'orange'">
                <svg><use xlink:href="#create--sprite"></use></svg>
                <span>Share</span>
            </Button>
        </div>
    </div>
</template>

<script>
/* eslint-disable */
import "@/assets/refresh-ccw.svg?sprite";
import "@/assets/plus-bold.svg?sprite";
import "@/assets/forward-arrow.svg?sprite";
import Loading from '@/components/Loading.vue';
import { EventBus } from '@/events/bus.js';
import { Swiper, SwiperSlide } from 'vue-awesome-swiper';
import Product from '@/components/product/ProductItem.vue';
import Post from '@/components/Post.vue';
import Share from '@/components/Share.vue';
import Button from '@/components/ui/Button.vue';
import Icon from '@/components/ui/Icon.vue';
import Avatar from '@/components/ui/Avatar.vue';
import TrendingProfile from '@/components/TrendingProfile.vue';
import urlencode from 'urlencode';
import 'swiper/css/swiper.css';

export default {
    name: 'Trending',
    data() {
        return {
            trending: '',
            cursor_products: '',
            cursor_posts: '',
            is_loading_posts: true,
            trending_profile: {},
            featured_creators: [],
            related_posts: [],
            is_sticky: false,
            offset: 48,
            trending_type: [{name: 'tt', desc:'App\\TikTokVideo'}, {name: 'ig', desc:'App\\InstagramMedia'}],
            creators_swiper: {
                slidesPerView: 'auto',
                watchOverflow: false,
                spaceBetween: 0,
                freeMode: true,
                navigation: {
                    nextEl: ".next-2",
                    prevEl: ".prev-2",
                }
            },
            requestURL: ''
        }
    },
    
    components: {
        Loading,
        Product,
        Post,
        Share,
        Button,
        Icon,
        Avatar,
        Swiper,
        SwiperSlide,
        TrendingProfile,
    },

    methods: {
        getTrendingProfile() {
            this.$ajax(`/api/trending-hashtag/profile/${this.trending_id}`, {
                method: 'get',
            }).then(res => {
                if (res.data.status === 'success') {
                    this.trending_profile = res.data.data;
                }
            });
        },

        getFeaturedCreators() {
            this.$ajax(`/api/trending-hashtag/feature-creators/${this.trending_id}`, {
                method: 'get',
            }).then(res => {
                if (res.data.status === 'success') {
                    this.featured_creators = res.data.data.data;
                }
            });
        },

        getRelatedPosts(page) {
            this.requestURL = `/api/trending-hashtag/posts/${this.trending_id}?page=${page}`;
            this.$ajax(`/api/trending-hashtag/posts/${this.trending_id}?page=${page}`, {
                method: 'get',
            }).then(res => {
                if (res.data.status === 'success') {
                    this.cursor_posts = (res.data.data.last_page > res.data.data.current_page) ? res.data.data.current_page : '';
                    res.data.data.data.forEach(item => {
                        if (this.related_posts.find(p => p.social_post_url === item.social_post_url))
                        {
                            if (item.post_author_type !== 2) {
                                if (this.related_posts.find(p => p.social_post_url === item.social_post_url)['post_author_type'] === 2) {
                                    this.related_posts.splice(this.related_posts.findIndex(p => p.social_post_url === item.social_post_url && p.post_author_type === 2),1)
                                } 
                                this.related_posts.push(item);
                            }
                        } else {
                            this.related_posts.push(item);
                        }
                        
                    });
                    this.is_loading_posts = false;
                }
            });
        },

        viewUser(id) {
            this.$router.push(`/user/${id}/profile`);
        },

        follow(user,index) {
            if (!this.$store.state.is_logged) {
                this.$cookie.set('follow_user', user.user_id, { expires: '15m' });

                this.$store.commit('login', {
                    is_show: true,
                    title: 'Follow ' + user.nickname + ' to see what they post or tag on Motom.',
                    text: 'Don’t want to miss their posts? Log in to your account and then hit follow or click the Sign up button at the bottom to create your Motom account.'
                });
                return;
            }
            this.$ajax('/api/post/user/'+user.user_id+'/follow', {
                method: 'post',
                withCredentials: true,
            }).then(res => {
                if (res && res.data && res.data.status === 'success') {
                    this.featured_creators[index].is_followed = true;
                }
            })
            .catch(err => {
                if (err.response.data.message === 'Unauthenticated.') {
                } 
            });
        },

        handleScroll () {
            // for Share Button
            const current_scroll_position = window.pageYOffset || document.documentElement.scrollTop;
            if (current_scroll_position > this.offset) {
                this.is_sticky = true;
            } else {
                this.is_sticky = false;
            }
            this.offset = current_scroll_position;


            // for next page
            let body = document.body;
            let html = document.documentElement;
            let docHeight = Math.max( body.scrollHeight, body.offsetHeight, html.clientHeight, html.scrollHeight, html.offsetHeight );
            let scrollY = Math.max(0, html.scrollTop || window.pageYOffset || 0);
            
            if (scrollY > (docHeight - (window.innerHeight * 1.7)) && !this.is_loading_posts && this.cursor_posts !== '') {
                this.is_loading_posts = true;
                this.getNextPagePosts(this.cursor_posts);
            }
        },

        getNextPagePosts() {
            this.getRelatedPosts(this.cursor_posts + 1);
        },

        share() {
            let obj = {};

            let label = window.user_id_alpha ? window.user_id_alpha : 'undefined';
            label = label  + ',' + this.trending_id + ',,';
            this.GA_Tracking('SHARE', 'CLICK', label);

            obj.shareURL = `${window.location.origin}${this.$route.fullPath}`;
            obj.postType = '';
            obj.twitter_text = 'Trending #'+ this.trending_profile.text +' on Motom: ' + obj.shareURL;
            obj.email_subject = 'Check out this Motom Trending!';
            obj.email_body = 'Trending #'+ this.trending_profile.text +' on Motom by clicking here: ' + obj.shareURL;

            EventBus.$emit('open-post-share', obj);
        },
    },

    mounted() {
        let encode_text = urlencode(this.$route.params.text);
        this.$ajax(`/api/trending-hashtag/profile/${this.$route.params.type === 'ig' ? 'in' : this.$route.params.type}/${encode_text}`, {
            method: 'get',
        }).then(res => {
            if (res.data.status === 'success') {
                this.trending_profile = res.data.data;
                this.trending_id = res.data.data.id;
                this.getFeaturedCreators();
                this.getRelatedPosts(1);
            }
        }).catch(err => {
            this.$router.replace('/');
        });
    },

    created () {
        window.addEventListener('scroll', this.handleScroll)
    },
    destroyed () {
        window.removeEventListener('scroll', this.handleScroll);
    },
}

</script>

<style lang="scss">
    @import '../resources/sass/_variables.scss';
    
    #mobile-wrap.page--trending {
        .container {
            overflow: hidden;
            background: var(--bg-06);

            .nav-menu {
                width: auto;
                position: relative;
                .nav {
                    position: absolute;
                    width: 32px;
                    height: 32px;
                    background: #fff;
                    border: 1px solid #F2F2F2;
                    box-sizing: border-box;
                    box-shadow: 2px 4px 30px 1px rgba(0, 0, 0, 0.15);
                    border-radius: 100px;
                    cursor: pointer;
                    z-index: 2;

                    .arrow {
                        cursor: pointer;
                        fill: var(--black-90);
                        height: 1.3rem;
                        width: 0.7rem;
                        align-items: center;
                        margin: auto 0;
                        margin: 0;
                        top: 0.8rem;
                        left: 1rem;
                        position: absolute;
                    }
                    .arrow-reverse {
                        transform: rotate(180deg);
                        left: unset;
                        right: 1rem;
                    }
                }
                
                .swiper-button-disabled {
                    display: none !important
                }

                .prev {
                    left: 10px;
                }

                .next {
                    right: 10px;
                }
            }

            .content {
                display: flex;
                flex-direction: column;
                align-items: flex-start;
                padding: 0px;

                .list-header {
                    font-weight: 600;
                    font-size: 16px;
                    line-height: 20px;
                    display: flex;
                    align-items: center;
                    text-align: center;
                    color: #000000;
                    flex: none;
                    order: 0;
                    flex-grow: 0;
                    margin: 11px 0px;
                }

                .list-1 {
                    margin: var(--spacing-05) 0 var(--spacing-05) var(--spacing-05);
                    width: calc(100% - var(--spacing-05));

                    .prev, .next {
                        margin-top: 30px;
                        display: unset;
                    }

                    .swiper-container {
                        width: calc(100vw - var(--spacing-05));
                    }

                    .swiper-wrapper,
                    .swiper-slide {
                        height: 100%;
                    }

                    .swiper-slide {
                        width: auto;
                        padding-right: 1.7rem;

                        &:last-child {
                            padding-right: 0rem;
                        }
                    }

                    .product-item {
                        width: 78px;

                        .box {
                            box-shadow: none;

                            img {
                                width: 78px;
                            }
                        }

                        & > a > span::before {
                            padding: 0 0 115%;
                        }
                    }
                }

                .list-2 {
                    width: 100%;
                    padding: var(--spacing-05);

                    .list {
                        display: grid;
                        grid-template-columns: repeat(auto-fit, calc(50% - (var(--spacing-03)/2)));
                        grid-gap: var(--spacing-03);
                        row-gap: var(--spacing-04);
                        padding: 0;
                        width: 100%;

                        .product-item {
                            .box {
                                padding: 6px;
                                border: 1px solid #F2F2F2;
                                a {
                                    border-radius: 6px;
                                }
                                .like-wrap {
                                    position: absolute;
                                    bottom: 6px;
                                    :not(.is-liked) {
                                        svg {
                                            fill: var(--black-50);
                                        }
                                    }
                                }
                                .product-name {
                                    white-space: unset;
                                    display: -webkit-box;
                                    -webkit-line-clamp: 2;
                                    -webkit-box-orient: vertical;
                                    overflow: hidden;
                                    height: 28px;
                                    font: var(--p-12);
                                    font-weight: var(--semibold);
                                    margin-bottom: 6px;
                                }
                            }
                        }
                    }
                }

                .list-3 {
                    margin: var(--spacing-05) 0 var(--spacing-05) var(--spacing-05);

                    .swiper-container {
                        width: 100vw;
                    }
                    
                    .swiper-slide {
                        width: 120px;
                        &:last-child {
                            margin-right: 4rem;
                        }
                    }
                    
                    .brand-logo {
                        position: relative;
                        cursor: pointer;
                        width: 64px;
                        height: 64px;
                        margin: 0 auto;

                        &::before {
                            content: "";
                            padding: 0 0 100%;
                        }
                        
                        .ui-avatar {
                            div {
                                // width: 6.8rem;
                                // height: 6.8rem;
                                background: linear-gradient(102.8deg, #C4302B 6.94%, #F0CB35 91.73%);
                                padding: 0.2rem;
                                border: unset !important;
                            }
                        }
                    }

                    .brand-name {
                        color: $dark_grey;
                        font: var(--p-14);
                        font-weight: var(--semibold);
                        text-align: center;
                        margin-top: 5px;
                        margin-bottom: 5px;
                        word-break: break-word;
                        height: 40px;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        max-width: 140px;
                        span {
                            display: -webkit-box;
                            -webkit-line-clamp: 2;
                            -webkit-box-orient: vertical;
                            overflow: hidden;
                            text-overflow: ellipsis;
                        }
                    }

                    .brand-btn {
                        display: flex;
                        margin-bottom: 20px;
                    }
                    .ui-button {
                        text-align: center;
                        position: relative;
                        font-weight: var(--semibold);
                        margin: 0 auto;

                        &.disabled {
                            background: #FFFFFF;
                            border: 1px solid #BDBDBD;
                            color: #58595B;
                        }
                    }

                    .list {
                        display: grid;
                        grid-template-columns: 140px 140px;
                        grid-gap: 0.8rem;
                    }
                }

                .list-4 {
                    padding: var(--spacing-03);
                    width: 100%;
                }

                .right {
                    display: flex;
                    justify-content: center;
                    width: 100%;
                    .list-3 {
                        margin: unset;
                        margin-left: 1.2rem;
                        background-color: white;
                        height: fit-content;
                        padding: 16px;
                        border-radius: 1.6rem;
                        .list-header {
                            .header-black {
                                width: 100%;
                            }
                        }
                        .list {
                            .card {
                                border: 1px solid #F2F2F2;
                                border-radius: 6px;
                                padding: 1.2rem;
                                .brand-btn {
                                    margin-bottom: unset;
                                }
                            }
                        }
                    }
                }
            }
        }

        .share-button {
            position: fixed;
            bottom: 2.2rem;
            left: 50%;
            z-index: 109;
            width: 34rem;
            margin-left: -17rem;
            transition: .2s transform ease-in-out;
            transform: translate(0, 0%);
            display: unset;
            svg {
                display: none;
            }
            .ui-button {
                margin: auto;
                display: block;
                text-align: center;
                box-shadow: 0px 4px 8px 0px #F1652959, 1px 2px 4px 0px #F9A32652;
                background-image: linear-gradient(var(--mbg-bttf));
                font-size: 1.6rem;
                font-weight: 600;
                width: 23rem;
                transition: .2s width ease-in-out;
            }
        }

        .is-sticky {
            &.share-button {
                transform: translateY(170%);
            }
        }
    }

    @media (min-width: 1221px) {
        #mobile-wrap.page--trending {
            .container {
                // background-color: violet;
                padding: 44px 43px 0 43px;
                .right {
                    margin-top: 32px;
                    .list-3 {
                        .list {
                            grid-template-columns: 140px 140px;
                        }
                    }
                    .list-4 {
                        width: var(--post-max-width);
                        padding: unset;
                        margin-right: 1.2rem;
                        // article {
                        //     margin: 0;
                        // }
                    }
                }
            }
        }
    }

    @media (min-width: $mobile_size) and (max-width: 1220px) {
        #mobile-wrap.page--trending {
            .container {
                // background-color: yellowgreen;
                .right {
                    .list-3 {
                        .list {
                            grid-template-columns: 1fr;
                        }
                    }
                    .list-4 {
                        width: var(--post-max-width);
                        padding: unset;
                        margin-right: 1.2rem;
                        // article {
                        //     margin: 0;
                        // }
                    }
                }
            }
        }
    }
</style>
