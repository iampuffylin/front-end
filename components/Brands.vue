<template>
<!-- eslint-disable -->
    <div class="brand-page">
        <div class="section" v-if="$store.state.is_logged && brands.length > 0">
            <div class="list-1">
                <!-- <Loading v-if="brands.length === 0" :classname="['static']" /> -->
                <div class="list-header">
                    <div class="header-grey">Brands You’re Following</div>
                    <div class="head-link" @click="editBrands"><span class="text">Edit</span></div>
                </div>
                <Swiper :options="product_swiper">
                    <SwiperSlide v-for="(item, index) in brands" :key="item.id +'-'+ index">
                        <div class="brand-logo" @click="viewShop(item.id)">
                            <Avatar :size="'large'" :image="item.image" :is_shaded="true" :is_brand="true" :is_brand_check="false" />
                            <!-- <img class="brand-logo" :style="{'background-image': 'url('+ (item.image == '') ? default_image : item.image  +')' }"></img> -->
                        </div>
                        <div class="brand-name"><span>{{ item.name }}</span></div>
                        <div class="brand-btn"><Button :type="'bg'" :size="'32'" :color="'white'" @click.native="viewShop(item.id)">View shop</Button></div>
                    </SwiperSlide>
                </Swiper>
            </div>
        </div>

        <div class="section list-2">
            <!-- <Loading v-if="brands_style.length === 0" :classname="['static']" /> -->
            <div class="list-header"><div class="header-grey">Shop Popular Brands</div></div>
            <div v-if="$store.state.type === 'mobile'">
                <div v-for="index in brands_style.map((_,i)=>i).filter(i=>i%2===0)">
                    <div class="brand-item-a">
                        <div class="box-follow-link">
                            <div v-if="brands_style[index].brand.is_followed == 0" class="follow-link" @click="follow(index)">
                                <span class="icon"><svg><use xlink:href="#plus-bold--sprite"></use></svg></span><span class="text">Follow</span>
                            </div>
                        </div>
                        <div class="nav-menu">
                            <div :class="'nav prev prev-'+brands_style[index].brand.id"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                            <div :class="'nav next next-'+brands_style[index].brand.id"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                        </div>
                        <Swiper :options="brands_style[index].product_swiper">
                            <SwiperSlide>
                                <div class="brand-header">
                                    <div class="brand-logo" @click="viewShop(brands_style[index].brand.id)">
                                        <Avatar :size="'large'" :image="brands_style[index].brand.image" :is_brand="true" :is_brand_check="false" />
                                        <!-- <img class="brand-logo" :style="{'background-image': 'url('+ brands_style[index].brand.image  +')' }"></img> -->
                                    </div>
                                    <div class="brand-name"><span>{{ brands_style[index].brand.name }}</span></div>
                                    <div class="brand-btn"><Button :type="'bg'" :color="'orange'" :size="'32'" @click.native="viewShop(brands_style[index].brand.id)">View shop</Button></div>
                                </div>
                                <div class="product-list">
                                    <Product v-for="(product, index) in brands_style[index].products.slice(0,4)" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                </div>
                            </SwiperSlide>
                        </Swiper>
                    </div>
                    <div v-if="index+1 < brands_style.length" class="brand-item-b">
                        <div class="box-follow-link">
                            <div v-if="brands_style[index+1].brand.is_followed == 0" class="follow-link" @click="follow(index+1)">
                                <span class="icon"><svg><use xlink:href="#plus-bold--sprite"></use></svg></span><span class="text">Follow</span>
                            </div>
                        </div>
                        <div class="nav-menu">
                            <div :class="'nav prev prev-'+brands_style[index+1].brand.id"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                            <div :class="'nav next next-'+brands_style[index+1].brand.id"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                        </div>
                        <Swiper :options="brands_style[index+1].product_swiper">
                            <SwiperSlide>
                                <div class="brand-header">
                                    <div class="brand-logo" @click="viewShop(brands_style[index+1].brand.id)">
                                        <Avatar :size="'large'" :image="brands_style[index+1].brand.image" :is_brand="true" :is_brand_check="false" />
                                        <!-- <img class="brand-logo" :style="{'background-image': 'url('+ brands_style[index+1].brand.image  +')' }"></img> -->
                                    </div>
                                    <div class="brand-name"><span>{{ brands_style[index+1].brand.name }}</span></div>
                                    <div class="brand-btn"><Button :type="'bg'" :color="'orange'" :size="'32'" @click.native="viewShop(brands_style[index+1].brand.id)">View shop</Button></div>
                                </div>
                                <div class="product-list">
                                    <div class="product-list-1">
                                        <Product v-for="(product, index) in brands_style[index+1].products.slice(0,Math.ceil(brands_style[index+1].products.length/2))" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                    </div>
                                    <div class="product-list-2">
                                        <Product v-for="(product, index) in brands_style[index+1].products.slice(Math.ceil(brands_style[index+1].products.length/2))" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                    </div>
                                </div>
                            </SwiperSlide>
                        </Swiper>
                    </div>
                </div>
            </div>
            <div v-else class="desktop">
                <div v-for="index in brands_style.map((_,i)=>i).filter(i=>i%4===0)">
                    <div class="left">
                        <div class="brand-item-a">
                            <div class="box-follow-link">
                                <div v-if="brands_style[index].brand.is_followed == 0" class="follow-link" @click="follow(index)">
                                    <span class="icon"><svg><use xlink:href="#plus-bold--sprite"></use></svg></span><span class="text">Follow</span>
                                </div>
                            </div>
                            <div class="nav-menu">
                                <div :class="'nav prev prev-'+brands_style[index].brand.id"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                                <div :class="'nav next next-'+brands_style[index].brand.id"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                            </div>
                            <Swiper :options="brands_style[index].product_swiper">
                                <SwiperSlide>
                                    <div class="brand-header">
                                        <div class="brand-logo" @click="viewShop(brands_style[index].brand.id)">
                                            <Avatar :size="'large'" :image="brands_style[index].brand.image" :is_brand="true" :is_brand_check="false" />
                                            <!-- <img class="brand-logo" :style="{'background-image': 'url('+ brands_style[index].brand.image  +')' }"></img> -->
                                        </div>
                                        <div class="brand-name"><span>{{ brands_style[index].brand.name }}</span></div>
                                        <div class="brand-btn"><Button :type="'bg'" :color="'orange'" :size="'32'" @click.native="viewShop(brands_style[index].brand.id)">View shop</Button></div>
                                    </div>
                                    <div class="product-list">
                                        <Product v-for="(product, index) in brands_style[index].products.slice(0,4)" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                    </div>
                                </SwiperSlide>
                            </Swiper>
                        </div>
                        <div v-if="(index+2) < brands_style.length" class="brand-item-b">
                            <div class="box-follow-link">
                                <div v-if="brands_style[index+2].brand.is_followed == 0" class="follow-link" @click="follow(index+2)">
                                    <span class="icon"><svg><use xlink:href="#plus-bold--sprite"></use></svg></span><span class="text">Follow</span>
                                </div>
                            </div>
                            <div class="nav-menu">
                                <div :class="'nav prev prev-'+brands_style[index+2].brand.id"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                                <div :class="'nav next next-'+brands_style[index+2].brand.id"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                            </div>
                            <Swiper :options="brands_style[index+2].product_swiper">
                                <SwiperSlide>
                                    <div class="brand-header">
                                        <div class="brand-logo" @click="viewShop(brands_style[index+2].brand.id)">
                                            <Avatar :size="'large'" :image="brands_style[index+2].brand.image" :is_brand="true" :is_brand_check="false" />
                                            <!-- <img class="brand-logo" :style="{'background-image': 'url('+ brands_style[index+2].brand.image  +')' }"></img> -->
                                        </div>
                                        <div class="brand-name"><span>{{ brands_style[index+2].brand.name }}</span></div>
                                        <div class="brand-btn"><Button :type="'bg'" :color="'orange'" :size="'32'" @click.native="viewShop(brands_style[index+2].brand.id)">View shop</Button></div>
                                    </div>
                                    <div class="product-list">
                                        <div class="product-list-1">
                                            <Product v-for="(product, index) in brands_style[index+2].products.slice(0,Math.ceil(brands_style[index+2].products.length/2))" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                        </div>
                                        <div class="product-list-2">
                                            <Product v-for="(product, index) in brands_style[index+2].products.slice(Math.ceil(brands_style[index+2].products.length/2))" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                        </div>
                                    </div>
                                </SwiperSlide>
                            </Swiper>
                        </div>
                    </div>
                    <div class="right">
                        <div v-if="(index+1) < brands_style.length" class="brand-item-b">
                            <div class="box-follow-link">
                                <div v-if="brands_style[index+1].brand.is_followed == 0" class="follow-link" @click="follow(index+1)">
                                    <span class="icon"><svg><use xlink:href="#plus-bold--sprite"></use></svg></span><span class="text">Follow</span>
                                </div>
                            </div>
                            <div class="nav-menu">
                                <div :class="'nav prev prev-'+brands_style[index+1].brand.id"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                                <div :class="'nav next next-'+brands_style[index+1].brand.id"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                            </div>
                            <Swiper :options="brands_style[index+1].product_swiper">
                                <SwiperSlide>
                                    <div class="brand-header">
                                        <div class="brand-logo" @click="viewShop(brands_style[index+1].brand.id)">
                                            <Avatar :size="'large'" :image="brands_style[index+1].brand.image" :is_brand="true" :is_brand_check="false" />
                                            <!-- <img class="brand-logo" :style="{'background-image': 'url('+ brands_style[index+1].brand.image  +')' }"></img> -->
                                        </div>
                                        <div class="brand-name"><span>{{ brands_style[index+1].brand.name }}</span></div>
                                        <div class="brand-btn"><Button :type="'bg'" :color="'orange'" :size="'32'" @click.native="viewShop(brands_style[index+1].brand.id)">View shop</Button></div>
                                    </div>
                                    <div class="product-list">
                                        <div class="product-list-1">
                                            <Product v-for="(product, index) in brands_style[index+1].products.slice(0,Math.ceil(brands_style[index+1].products.length/2))" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                        </div>
                                        <div class="product-list-2">
                                            <Product v-for="(product, index) in brands_style[index+1].products.slice(Math.ceil(brands_style[index+1].products.length/2))" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                        </div>
                                    </div>
                                </SwiperSlide>
                            </Swiper>
                        </div>
                        <div v-if="(index+3) < brands_style.length" class="brand-item-a">
                            <div class="box-follow-link">
                                <div v-if="brands_style[index+3].brand.is_followed == 0" class="follow-link" @click="follow(index+3)">
                                    <span class="icon"><svg><use xlink:href="#plus-bold--sprite"></use></svg></span><span class="text">Follow</span>
                                </div>
                            </div>
                            <div class="nav-menu">
                                <div :class="'nav prev prev-'+brands_style[index+3].brand.id"><svg class="arrow"><use :xlink:href="'#back--sprite'"></use></svg></div>
                                <div :class="'nav next next-'+brands_style[index+3].brand.id"><svg class="arrow arrow-reverse"><use :xlink:href="'#back--sprite'"></use></svg></div>
                            </div>
                            <Swiper :options="brands_style[index+3].product_swiper">
                                <SwiperSlide>
                                    <div class="brand-header">
                                        <div class="brand-logo" @click="viewShop(brands_style[index+3].brand.id)">
                                            <Avatar :size="'large'" :image="brands_style[index+3].brand.image" :is_brand="true" :is_brand_check="false" />
                                            <!-- <img class="brand-logo" :style="{'background-image': 'url('+ brands_style[index+3].brand.image  +')' }"></img> -->
                                        </div>
                                        <div class="brand-name"><span>{{ brands_style[index+3].brand.name }}</span></div>
                                        <div class="brand-btn"><Button :type="'bg'" :color="'orange'" :size="'32'" @click.native="viewShop(brands_style[index+3].brand.id)">View shop</Button></div>
                                    </div>
                                    <div class="product-list">
                                        <Product v-for="(product, index) in brands_style[index+3].products.slice(0,4)" :product="product" :key="product.id +'-'+ index" :hide_like_button="true"/>
                                    </div>
                                </SwiperSlide>
                            </Swiper>
                        </div>
                    </div>
                </div>
            </div>

            <Loading v-if="is_loading_products" :classname="['static']" :requestURL="requestURL"/>
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
import Product from '@/components/product/SearchProductItem.vue';
import shuffle from '@/utils/Shuffle';
import Button from '@/components/ui/Button.vue';
import Avatar from '@/components/ui/Avatar.vue';
import 'swiper/css/swiper.css';

export default {
    name: 'Shop',
    data() {
        return {
            brands: [],
            products: [],
            pop_brands: [],
            brands_style: [],
            is_loading_products: true,
            is_switch: false,
            ajax_url: '',
            parent_index: false,
            on_sale: '',
            cursor: '',
            order_by: '',
            text: '',
            price_from: 1,
            price_to: 500,
            product_swiper: {
                slidesPerView: 'auto',
                watchOverflow: false,
                spaceBetween: 0,
                freeMode: true,
                navigation: {
                    nextEl: ".next",
                    prevEl: ".prev",
                }
            },
            requestURL: '',
        }
    },
    
    components: {
        Loading,
        Product,
        Button,
        Avatar,
        Swiper,
        SwiperSlide,
    },

    methods: {
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
                    text: "Log in to save this post to a LikeBoard. If you don't have a Motom account yet, click 'Sign up' below."
                });
            }
        },
        
        handleAjaxError(error) {
            this.$store.state.show_loading = false;
            console.log(error)
        },

        getNextPageBrands() {
            this.getBrandsStyle(this.cursor + 1);
        },

        getFollowingBrands() {
            this.$ajax.get('/api/brands/following', {
                method: 'get',
                withCredentials: true,
            }).then(res => {
                let data = res.data.data;

                let i = data.length, k , temp; // k is to generate random index and temp is to swap the values
                while(--i > 0){
                    k = Math.floor(Math.random() * (i+1));
                    temp = data[k];
                    data[k] = data[i];
                    data[i] = temp;
                }

                let numShow = data.length;
                if (numShow > 6) {
                    numShow = 6;
                }
                
                let the_list = [];
                // some brands have no logo
                for (let j = 0; j < numShow; j++) {
                    if (data[j].image !== '') {
                        data[j].image = data[j].image;
                    } else {
                        data[j].image = '/assets/tmp/cate/default.svg';
                    }

                    the_list.push(data[j]);
                }                

                // assign to internal data
                this.brands = the_list;
                
            })            
            .catch(this.handleAjaxError);
        },

        getBrandsStyle(page) {
            let self = this;
            this.requestURL = '/api/shop/shop-the-brands?page=' + page;
            this.$ajax.get('/api/shop/shop-the-brands?page=' + page, {
                method: 'get',
                withCredentials: true,
            }).then(res => {
                this.cursor = '';
                let data = res.data.data.data;
                if (res.data.data.current_page < res.data.data.last_page) {
                    self.cursor = res.data.data.current_page;
                } else {
                    self.cursor = '';
                }
                
                let the_list = [];
                // some brands have no logo
                for (let j = 0; j < data.length; j++) {
                    if (data[j].image !== '') {
                        data[j].image = data[j].image;
                    } else {
                        data[j].image = '/assets/tmp/cate/default.svg';
                    }

                    the_list.push(data[j]);
                }
                for (let i = 0; i < the_list.length; i++) {
                    let product_swiper = {
                        slidesPerView: 'auto',
                        watchOverflow: false,
                        spaceBetween: 0,
                        freeMode: true,
                        navigation: {
                            nextEl: ".next-" + the_list[i].brand.id,
                            prevEl: ".prev-" + the_list[i].brand.id,
                        }
                    }
                    the_list[i].product_swiper = product_swiper;
                    the_list[i].products = the_list[i].products.slice(0,10);

                    self.brands_style.push(the_list[i]);
                }
                this.is_loading_products = false;
            })            
            .catch(this.handleAjaxError);
            
        },

        viewShop(id) {
            window.location.href = '/brand/'+ id +'/profile';
        },

        editBrands() {
            window.location.href = '/me/settings/myBrands';
        },

        follow(index) {
            if (!this.$store.state.is_logged) {
                this.$cookie.set('follow_brand', this.brands_style[index].brand.id, { expires: '15m' });

                this.$store.commit('login', {
                    is_show: true,
                    title: 'Follow ' + this.brands_style[index].brand.name + ' to see what they post or tag on Motom.',
                    text: 'Don’t want to miss their posts? Log in to your account and then hit follow or click the Sign up button at the bottom to create your Motom account.'
                });
                return;
            }
            this.$ajax('/api/post/brand/'+this.brands_style[index].brand.id+'/watch', {
                method: 'post',
                withCredentials: true,
            }).then(res => {
                if (res && res.data && res.data.status === 'success') {
                    this.brands_style[index].brand.is_followed = 1;
                    this.brands.push(this.brands_style[index].brand);
                }
            })
            .catch(err => {
                if (err.response.data.message === 'Unauthenticated.') {
                } 
  
            });
        },

        handleScroll() {
            let body = document.body;
            let html = document.documentElement;
            let docHeight = Math.max( body.scrollHeight, body.offsetHeight, html.clientHeight, html.scrollHeight, html.offsetHeight );
            let scrollY = Math.max(0, html.scrollTop || window.pageYOffset || 0);
            console.log(this.cursor);
            if (scrollY > (docHeight - (window.innerHeight * 1.7)) && !this.is_loading_products && this.cursor !== '') {
                this.is_loading_products = true;
                this.getNextPageBrands(this.cursor);
            }
        },
    },

    mounted() {
        this.getFollowingBrands();
        this.getBrandsStyle(1);
    },

    created () {
        window.addEventListener('scroll', this.handleScroll);
    },

    destroyed () {
        window.removeEventListener('scroll', this.handleScroll);
    },
}

</script>

<style lang="scss">
    @import '../resources/sass/_variables.scss';
    
    .brand-page {
        overflow: hidden;
        background: var(--bg-06);

        .list-header {
            margin-bottom: var(--spacing-05);
        }

        .header {
            &-white {
                color: $white;
                font: var(--h4);
                font-weight: var(--semibold);
            }
            &-grey {
                color: $dark_grey;
                font: var(--h4);
                font-weight: var(--semibold);
            }
            &-black {
                color: $black;
                font: var(--h4);
                font-weight: var(--semibold);
            }
        }

        .head-link {
            cursor: pointer;
            color: $med_grey;
            position: relative;
            top: -20px;
            float: right;
            font: var(--p-14);
        }

        .icon {
            position: absolute;
            display: inline-block;
            top: 10px;
            right: 100px;

            svg {
                height: 13.33px;
                width: 10.67px;
                position: absolute;
                left: 50%;
                top: 50%;
                transform: translate(-50%, -50%);
                fill: white;
            }
        }

        .text {
            font-weight: var(--medium);
            white-space:nowrap;
        }

        .product-item {
            background: transparent;
            box-shadow: none;
            border: none;

            .box {
                box-shadow: 0px 2px 0px rgba(black, 0.04);
                height: unset;
            }

            &.img-200 {
                .like-wrap {
                    top: unset;
                    left: unset;
                    bottom: 5px;
                    right: 5px;
                }
            }

            &.img-400 {
                .like-wrap {
                    top: unset;
                    left: unset;
                    bottom: 100px;
                    right: 30px;
                }
            }

            a {
                border-radius: 6px;
            }

            .like {
                box-shadow: 2px 4px 20px rgba(#000, .1);
                border: 1px solid $grey_white;
                margin: 14px;
            }
        }

        .section {
            & > header {
                margin: $padding;
                display: flex;
                align-items: center;

                h2 {
                    font-weight: var(--semibold);
                    font: var(--h6);
                    flex: 1;
                }

                a {
                    font: var(--p-12);
                    font-weight: var(--semibold);
                    color: $med_grey;
                }
            }
        }
        
        .box {
            width: 100%;
            padding: .5rem 0 2rem;
            position: relative;
            margin-top: -.5rem;
        }

        .brand-box-color {
            padding: 2rem 0 2rem 2rem;
            background-color: white;
            border-radius: 16px;
            margin: 0.5rem 0 2rem;
        }

        .list-1 {
            margin: var(--spacing-05) 0 var(--spacing-05) var(--spacing-05);
            margin-right: var(--spacing-05);

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
            }
        }

        .list-2 {
            margin: var(--spacing-05) 0 var(--spacing-11) 0;
            padding: var(--spacing-05) 0 var(--spacing-05) var(--spacing-05);
            background: $white;
            border-radius: 16px;
            background-color: white;

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
            }
            
            .swiper-button-disabled {
                display: none;
            }

            .nav-menu {
                width: auto;
                position: relative;
            }

            .prev {
                margin-top: 60px;
                margin-left: 10px;
            }

            .next {
                margin-top: 60px;
                margin-left: calc(100vw - 58px);
            }

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
            
            .swiper-container {
                width: 100%;
            }

            .swiper-wrapper,
            .swiper-slide {
                height: 100%;
                width: auto;
                padding: .6rem;
                display: flex;

                &:last-child {
                    padding-right: 2rem;
                }
            }

            .brand-item-a {
                background: linear-gradient(136.42deg, white -1.37%, #FFF6D6 43.74%, #F0F0F0 92.76%), white;
                // border: 1px solid white;
                box-sizing: border-box;
                // box-shadow: var(--bot-2dp);
                height: 225px;
                border-radius: 7px;
                margin-bottom: 30px;

                .brand-header {
                    width: 180px;
                    margin-right: 1.6rem;
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
                    }

                    .brand-name {
                        color: $dark_grey;
                        font: var(--p-14);
                        font-weight: var(--semibold);
                        text-align: center;
                        margin: 0 0.8rem 0.8rem 0.8rem;
                        word-break: break-word;
                        height: 40px;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        span {
                            display: -webkit-box;
                            -webkit-line-clamp: 2;
                            -webkit-box-orient: vertical;
                            overflow: hidden;
                            text-overflow: ellipsis;
                        }
                    }

                    .ui-avatar div {
                        border: unset;
                        padding: 0.2rem;
                        background: linear-gradient(102.8deg, #C4302B 6.94%, #F0CB35 91.73%);
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
                    }
                }

                .product-list {
                    .product-item {
                        width: 146px;
                        display: inline-block;
                        padding: 1.6rem 0.8rem 1.6rem 0;
                    }
                }

                .box-follow-link {
                    height: 40px;
                    .follow-link {
                        cursor: pointer;
                        padding-top: 20px;
                        float: right;
                        padding-right: 20px;
                        width: 90px;
                        height: 40px;
                        display: grid;
                        grid-template-columns: 25% 75%;
                    
                        .icon {
                            position: static;
                            // margin-left: 10px;
                            // vertical-align: middle;
                            display: flex;
                            align-items: center;
                            svg {
                                fill: $orange;
                                position: static;
                                transform: none;
                            }
                        }
                        .text {
                            color: $orange;
                            // margin-left: 6px;
                        }
                    }
                }
            }

            .brand-item-b {
                background: linear-gradient(103.37deg, #F4F9FF 24.6%, #CCDCF3 64.65%, #F4F9FF 99.18%);
                // border: 1px solid white;
                box-sizing: border-box;
                // box-shadow: var(--bot-2dp);
                height: 225px;
                border-radius: 7px;
                margin-bottom: 30px;

                .brand-header {
                    width: 180px;
                    margin-right: 1.6rem;
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
                    }

                    .brand-name {
                        color: $dark_grey;
                        font: var(--p-14);
                        font-weight: var(--semibold);
                        text-align: center;
                        margin: 0 0.8rem 0.8rem 0.8rem;
                        word-break: break-word;
                        height: 40px;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        span {
                            display: -webkit-box;
                            -webkit-line-clamp: 2;
                            -webkit-box-orient: vertical;
                            overflow: hidden;
                            text-overflow: ellipsis;
                        }
                    }

                    .ui-avatar div {
                        border: unset;
                        padding: 0.2rem;
                        background: linear-gradient(102.8deg, #C4302B 6.94%, #F0CB35 91.73%);
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

                        div {
                            border: unset;
                            padding: 0.2rem;
                            background: linear-gradient(102.8deg, #C4302B 6.94%, #F0CB35 91.73%);
                        }
                    }
                }

                .product-list-1 {
                    top: 20px;
                    .product-item {
                        width: 75px;
                        height: 75px;
                        display: inline-block;
                        padding: 0 8px 8px 0;
                    }
                }

                .product-list-2 {
                    height: 85px;
                    top: 20px;
                    .product-item {
                        width: 75px;
                        height: 75px;
                        display: inline-block;
                        padding: 0 8px 0 0;
                    }
                }

                .box-follow-link {
                    height: 40px;
                    .follow-link {
                        cursor: pointer;
                        padding-top: 20px;
                        float: right;
                        padding-right: 20px;
                        width: 90px;
                        height: 40px;
                        display: grid;
                        grid-template-columns: 25% 75%;
                    
                        .icon {
                            position: static;
                            // margin-left: 10px;
                            // vertical-align: middle;
                            display: flex;
                            align-items: center;
                            svg {
                                fill: $orange;
                                position: static;
                                transform: none;
                            }
                        }
                        .text {
                            color: $orange;
                            // margin-left: 6px;
                        }
                    }
                }
            }
        }
    }

    @media (min-width: $mobile_size) {
        .page--globalsearch .home-content {
            max-width: var(--modal-max-width);
        }

        .home-content {
            max-width: unset;

            .brand-page {
                padding: var(--spacing-07);
                min-height: calc(100vh - 96px);

                .list-2 {
                    padding: var(--spacing-05);

                    .next {
                        right: 10px;
                    }
                }

                .desktop {
                    display: inline-block;
                    .left {
                        width: calc((100% - 20px)/2);
                        float: left;
                    }
                    .right {
                        width: calc((100% - 20px)/2);
                        float: left;
                        margin-left: 20px;
                    }
                }
            }
        }
    }
</style>
