<template>
    <div>
        <div class="header-card">
            <div class="hashtag-header" :class="$route.params.type">
                <div class="head">
                    <div class="trend-logo" :class="$route.params.type">
                        <div class="logo-border">
                            <Icon v-if="$route.params.type === 'tt'" :type="'tikTok'" :icon="'tiktok-no-circle'" :size="'big'" :shape="'circle'" />
                            <Icon v-else-if="$route.params.type === 'ig'" :type="'instagram'" :icon="'instagram-no-circle'" :size="'big'" :shape="'circle'" />
                        </div>
                    </div>
                    <div class="main-text">
                        <div v-if="$route.params.type === 'tt'" class="tips">Shop products tagged to TikTok posts using the hashtag:</div>
                        <div v-else-if="$route.params.type === 'ig'" class="tips">Shop products tagged to Instagram posts using the hashtag:</div>
                        <div class="name">#{{ $route.params.text }}</div>
                    </div>
                </div>
                <div v-if="$route.params.type === 'tt'" class="desc">When you create a Motom post using a TikTok with this hashtag, the post and products will appear here!</div>
                <div v-else-if="$route.params.type === 'ig'" class="desc">When you create a Motom post using an Instagram with this hashtag, the post and products will appear here!</div>
            </div>
            <div class="trending-tab-nav">
                <Navi :main="menus" />
            </div>
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
import shuffle from '@/utils/Shuffle';
import Button from '@/components/ui/Button.vue';
import Icon from '@/components/ui/Icon.vue';
import Avatar from '@/components/ui/Avatar.vue';
import Navi from '@/components/Navi.vue';
import urlencode from 'urlencode';
import 'swiper/css/swiper.css';

export default {
    name: 'TrendingProfile',
    data() {
        return {
            offset: 48,
            menus: [],
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
        Navi,
        Swiper,
        SwiperSlide,
    },

    props: {
    },

    methods: {
    },

    mounted() {
        let encode_text = urlencode(this.$route.params.text);
        this.menus = [
            {
                text: 'Products',
                link: `/trending/product/${this.$route.params.type}/${encode_text}`
            },
            {
                text: 'Related Posts',
                link: `/trending/post/${this.$route.params.type}/${encode_text}`
            },
        ];
    },

    created () {
    },
}

</script>

<style lang="scss">
    @import '../resources/sass/_variables.scss';
    
    #mobile-wrap.page--trending {
        .header-card {
            box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
            border-radius: 1.6rem;
            margin-bottom: 3px;
            background: white;
            .hashtag-header {
                display: flex;
                flex-direction: column;
                align-items: flex-start;
                padding: 24px;
                position: static;
                left: 0%;
                right: 0%;
                top: 23.23%;
                bottom: 16.77%;
                background-blend-mode: normal, normal, multiply, normal;
                border-radius: 0px 0px 16px 16px;
                flex: none;
                order: 2;
                align-self: stretch;
                flex-grow: 0;
                margin: 0px 0px;

                &.tt {
                    background: linear-gradient(127.77deg, #000000 -4.95%, rgba(255, 24, 135, 0.28) 87.68%), linear-gradient(180deg, rgba(7, 73, 242, 0.65) 0%, rgba(0, 0, 0, 0) 100%), linear-gradient(133.42deg, #A3A3A3 -1.44%, #16DD7E 93.6%), #FFFFFF;
                }

                &.ig {
                    background: linear-gradient(115.13deg, rgba(0, 0, 0, 0) 1.18%, rgba(254, 44, 85, 0.33) 25.66%, rgba(254, 44, 85, 0.93) 57%, rgba(254, 44, 85, 0.33) 93.24%, rgba(37, 244, 238, 0) 151.02%), linear-gradient(133.42deg, #FF12BD -1.44%, #F4D325 93.6%), #FFFFFF;;
                }

                .head {
                    display: flex;
                    flex-direction: row;
                    align-items: center;
                    padding: 0px;

                    position: static;
                    width: 100%;
                    height: 68px;
                    left: 16px;
                    top: 24px;
                    flex: none;
                    order: 0;
                    flex-grow: 0;
                    margin: 16px 0px;
                    .trend-logo {
                        display: flex;
                        flex-direction: column;
                        align-items: center;
                        padding: 12px;
                        position: static;
                        height: 68px;
                        left: 0px;
                        right: 260px;
                        top: 0px;
                        background-blend-mode: normal, normal, multiply, normal;
                        border-radius: 8px;
                        flex: none;
                        order: 0;
                        flex-grow: 0;

                        &.tt {
                            background: linear-gradient(127.77deg, #000000 -4.95%, rgba(255, 24, 135, 0.28) 87.68%), linear-gradient(180deg, rgba(7, 73, 242, 0.65) 0%, rgba(0, 0, 0, 0) 100%), linear-gradient(133.42deg, #A3A3A3 -1.44%, #16DD7E 93.6%), #FFFFFF;
                        }

                        &.ig {
                            background: linear-gradient(115.13deg, rgba(0, 0, 0, 0) 1.18%, rgba(254, 44, 85, 0.33) 25.66%, rgba(254, 44, 85, 0.93) 57%, rgba(254, 44, 85, 0.33) 93.24%, rgba(37, 244, 238, 0) 151.02%), linear-gradient(133.42deg, #FF12BD -1.44%, #F4D325 93.6%), #FFFFFF;
                        }

                        .logo-border {
                            width: 46px;
                            height: 46px;
                            padding: 0.2rem;
                            background: linear-gradient(102.8deg, #C4302B 6.94%, #F0CB35 91.73%);
                            border-radius: 50%;
                            i {
                                width: 42px;
                                height: 42px;
                            }
                        }
                    }
                    .main-text {
                        display: flex;
                        flex-direction: column;
                        justify-content: center;
                        align-items: flex-start;
                        padding: 0px;

                        position: static;
                        left: 24.39%;
                        right: 0%;
                        top: 17.65%;
                        bottom: 17.65%;
                        // flex: none;
                        order: 1;
                        flex-grow: 1;
                        margin: 0px 12px;
                        .tips {
                            position: static;
                            left: 0%;
                            right: 0%;
                            top: 0%;
                            bottom: 68.18%;
                            font-family: Montserrat;
                            font-style: normal;
                            font-weight: 500;
                            font-size: 12px;
                            line-height: 14px;
                            display: flex;
                            align-items: center;
                            color: #E0E0E0;
                            flex: none;
                            order: 0;
                            align-self: stretch;
                            flex-grow: 0;
                            margin: 6px 0px;
                        }
                        .name {
                            position: static;
                            width: 164px;
                            height: 24px;
                            left: 0px;
                            top: 20px;
                            font-family: Montserrat;
                            font-style: normal;
                            font-weight: 600;
                            font-size: 20px;
                            line-height: 24px;
                            display: flex;
                            align-items: center;
                            letter-spacing: 0.15px;
                            color: #FFFFFF;
                            flex: none;
                            order: 1;
                            flex-grow: 0;
                            margin: 6px 0px;
                        }
                    }
                }
                .desc {
                    position: static;
                    left: 0%;
                    right: 0%;
                    top: 0%;
                    bottom: 0%;
                    font-family: Montserrat;
                    font-style: normal;
                    font-weight: 500;
                    font-size: 14px;
                    line-height: 18px;
                    display: flex;
                    align-items: center;
                    color: #FFFFFF;
                    flex: none;
                    order: 0;
                    align-self: stretch;
                    flex-grow: 0;
                    margin: 4px 0px;
                }
            }

            .trending-tab-nav {
                display: flex;
                flex-direction: row;
                justify-content: center;
                align-items: center;
                position: static;
                // width: 375px;
                height: 52px;
                background: #FFFFFF;
                flex: none;
                border-radius: 0px 0px 16px 16px;

                .site-navbar {
                    box-shadow: none !important;
                    background-color: unset !important;
                    justify-content: center !important;
                }

                .tab {
                    width: calc(50% - 40px);
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    align-items: center;
                    // padding: 0px 10px;
                    flex: none;
                    flex-grow: 1;
                    font-weight: 600;
                    font-size: 16px;
                    line-height: 20px;
                    display: flex;
                    align-items: center;
                    text-align: center;
                    color: #828282;
                    flex-grow: 0;
                    margin: 0px 10px;
                    display: inline-block;
                    cursor: pointer;
                    &.active {
                        color: #F15A29;
                        border-bottom:3px #F15A29 solid;
                    }
                    // span {
                    //     font-size: 12px;
                    //     font-weight: 500;
                    // }
                }
            }
        }
    }

    @media (min-width: $mobile_size) {
        #mobile-wrap.page--trending {
            .container {
                // margin-left: var(--sidebar-width);
                padding: 44px 43px 0 43px;
                .hashtag-header {
                    border-radius: 16px !important;
                }
                .list-1 {
                    margin: 16px !important;
                }
                .list-2 {
                    padding: unset;
                    .list {
                        grid-template-columns: minmax(150px,1fr) minmax(150px,1fr) minmax(150px,1fr) minmax(150px,1fr) !important;
                    }
                }
                .right {
                    margin-top: 32px;
                }
                .share-button {
                    display: none;
                }
            }
        }
    }
</style>
