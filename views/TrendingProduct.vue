<template>
    <div class="container">
        <TrendingProfile></TrendingProfile>
        <div class="content">
            <div class="list-1">
                <!-- <Loading v-if="continue_shopping.length === 0" :classname="['static']" /> -->
                <div class="list-header">
                    Hot products seen with this hashtag
                </div>
                <div class="nav-menu">
                    <div class="nav prev prev-1"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                    <div class="nav next next-1"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                </div>
                <Swiper :options="products_swiper">
                    <SwiperSlide v-for="(product, index) in hot_products" :key="product.id +'-'+ index">
                        <Product :product="product" :btn_type="'Like'" />
                    </SwiperSlide>
                </Swiper>
            </div>
            <div class="list-2">
                <div class="list-header"><div class="header-black">Shop #{{trending_profile.text}}</div></div>
                <div class="list">
                    <Product v-for="(product, index) in more_products" :product="product" :key="product.id +'-'+ index" :image_size="'400x400'" :show_seen_in="true" :show_info="true"  :btn_type="'Like'"/>
                </div>
            </div>
        </div>

        <Loading v-if="is_loading_products" :classname="['static']" :requestURL="requestURL"/>

        <Share :title="'Check out this Motom Trending!'" ref="share" @copied="shareCallback($event)" />

        <div class="share-button" @click="share" :class="{'is-sticky': is_sticky}">
            <Button :size="'40'" :type="'bg'" :color="'orange'">
                <svg><use xlink:href="#create--sprite"></use></svg>
                <span>Share</span>
            </Button>
        </div>

        <ProductPopup />
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
import Post from '@/components/Post.vue';
import Share from '@/components/Share.vue';
import Button from '@/components/ui/Button.vue';
import Icon from '@/components/ui/Icon.vue';
import Avatar from '@/components/ui/Avatar.vue';
import TrendingProfile from '@/components/TrendingProfile.vue';
import urlencode from 'urlencode';
import 'swiper/css/swiper.css';
import Product from '@/components/product/ProfileProductItem.vue';
import ProductPopup from '@/components/product/ProfileProductItemPopup.vue';

export default {
    name: 'Trending',
    data() {
        return {
            trending: '',
            cursor_products: '',
            cursor_posts: '',
            is_loading_products: true,
            trending_profile: {},
            // trending_count: [],
            // product_count: 0,
            // posts_count: 0,
            hot_products: [],
            more_products: [],
            is_sticky: false,
            offset: 48,
            trending_type: [{name: 'tt', desc:'App\\TikTokVideo'}, {name: 'ig', desc:'App\\InstagramMedia'}],
            products_swiper: {
                slidesPerView: 'auto',
                watchOverflow: false,
                spaceBetween: 0,
                freeMode: true,
                navigation: {
                    nextEl: ".next-1",
                    prevEl: ".prev-1",
                }
            },
            hashtag_id: 0,
            requestURL: '',
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
        ProductPopup,
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

        getHotProducts() {
            this.$ajax(`/api/trending-hashtag/hot-products/${this.trending_id}`, {
                method: 'get',
            }).then(res => {
                if (res.data.status === 'success') {
                    this.hot_products = res.data.data.data;
                    this.hot_products.forEach(hp => {
                        hp.hashtag_id = this.hashtag_id
                    })
                }
            });
        },

         getMoreProducts(page) {
            this.requestURL = `/api/trending-hashtag/products/${this.trending_id}?page=${page}`;
            this.$ajax(`/api/trending-hashtag/products/${this.trending_id}?page=${page}`, {
                method: 'get',
            }).then(res => {
                if (res.data.status === 'success') {
                    this.cursor_products = (res.data.data.last_page > res.data.data.current_page) ? res.data.data.current_page : '';
                    res.data.data.data.forEach(item => {
                        item.hashtag_id = this.hashtag_id;
                        this.more_products.push(item);
                    });
                    this.is_loading_products = false;
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
            
            if (scrollY > (docHeight - (window.innerHeight * 1.7)) && !this.is_loading_products && this.cursor_products !== '') {
                this.is_loading_products = true;
                this.getNextPageProducts(this.cursor_products);
            }
        },

        getNextPageProducts() {
            this.getMoreProducts(this.cursor_products + 1);
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
                this.hashtag_id = res.data.data.hashtag_text_id;
                this.getHotProducts();
                this.getMoreProducts(1);
            }
        }).catch(err => {
            // this.$router.replace('/');
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
                // position: absolute;
                // width: 376px;

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
                    // margin-right: var(--spacing-05);

                    .prev, .next {
                        margin-top: calc(78px/2 - 32px/2);
                        display: unset;
                    }

                    .swiper-container {
                        width: 100%; //calc(100vw - var(--spacing-05));
                    }

                    .swiper-wrapper {
                        height: 100%;
                        padding: unset !important;
                    }

                    .swiper-slide {
                        width: auto;
                        padding-right: 1.2rem !important;
                        height: 100%;

                        &:last-child {
                            padding-right: 0rem;
                        }
                    }

                    .product-item {
                        width: 78px;

                        .box {
                            box-shadow: none;

                            a {
                                border-radius: 6px;
                            }

                            img {
                                width: 78px;
                            }
                        }

                        .product-card-more-menu {
                            display: none;
                        }

                        & > a > span::before {
                            padding: 0 0 115%;
                        }
                    }
                }

                .list-2 {
                    // margin: var(--spacing-05) 0 var(--spacing-11) 0;
                    width: 100%;
                    padding: var(--spacing-05);
                    // border-radius: 16px;

                    .list {
                        display: grid;
                        grid-template-columns: repeat(auto-fit, calc(50% - (var(--spacing-03)/2)));
                        grid-gap: var(--spacing-03);
                        row-gap: var(--spacing-04);
                        padding: 0;
                        width: 100%;

                        .product-item {
                            .box {
                                padding: 0;
                                transition: .1s all ease;

                                & > a {
                                    border: none;
                                    border-radius: 0;
                                }
                            }

                            .info {
                                padding: 1.2rem;

                                .price {
                                    padding-right: 5rem;
                                }
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
                    // & > a > span::before {
                    //     padding: 0 0 115%;
                    // }
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

    @media (min-width: $mobile_size) {
        #mobile-wrap.page--trending {
            .container {
                // margin-left: var(--sidebar-width);
                padding: 44px 43px 0 43px;
                .list-1 {
                    margin: 16px !important;
                }
                .list-2 {
                    padding: unset;
                    .list {
                        grid-template-columns: minmax(150px,1fr) minmax(150px,1fr) minmax(150px,1fr) minmax(150px,1fr) !important;
                    }
                }
            }
        }
    }
</style>
