<template>
    <div class="container">
        <header class="search-header">
            <SearchBar />
            <Navi :main="menus" />
        </header>

        <div class="total-results" v-if="
            !is_loading_posts &&
            (
                selected_item ==='people' && people_count !== 0 ||
                selected_item ==='products' && product_count !== 0 ||
                selected_item ==='posts' && post_count !== 0 ||
                selected_item ==='hashtags' && hashtag_count !== 0 ||
                selected_item ==='brands' && brand_count !== 0 ||
                selected_item ==='likeboards' && likeboard_count !== 0
            )
        ">
            <p class="result-text">
                <strong v-if="selected_item ==='all'">{{ people_count }} </strong>
                <strong v-else-if="selected_item ==='people'">{{ people_count }} </strong>
                <strong v-else-if="selected_item ==='products'">{{ product_count }} </strong>
                <strong v-else-if="selected_item ==='posts'">{{ post_count }} </strong>
                <strong v-else-if="selected_item ==='hashtags'">{{ hashtag_count }} </strong>
                <strong v-else-if="selected_item ==='brands'">{{ brand_count }} </strong>
                <strong v-else-if="selected_item ==='likeboards'">{{ likeboard_count }} </strong>
                <span>Results</span>
            </p>

            <p class="small-text">Not seeing what you want? Try being more specific.</p>
        </div>

        <div v-else-if="is_loading_posts || is_loading_next" class="spinner-text">
            <span class="spinner-description">Hang tight - we're looking...</span>
        </div>

        <section class="search-content" v-if="!is_loading_posts">
            <div class="zero-results" v-if="
                (selected_item === 'top' && product_count === 0 && post_count === 0 && people_count === 0 && hashtag_count === 0 && likeboard_count === 0 && brand_count === 0 ) ||
                (selected_item === 'products' && product_count === 0 ) ||
                (selected_item === 'posts' && post_count === 0 ) ||
                (selected_item === 'people' && people_count === 0 ) ||
                (selected_item === 'hashtags' && hashtag_count === 0 ) ||
                (selected_item === 'likeboards' && likeboard_count === 0 ) ||
                (selected_item === 'brands' && brand_count === 0 )
            ">
                <NotFound />

                <div class="btns" v-if="
                    (selected_item !== 'products' && product_count !== 0 ) ||
                    (selected_item !== 'posts' && post_count !== 0 ) ||
                    (selected_item !== 'people' && people_count !== 0 ) ||
                    (selected_item !== 'hashtags' && hashtag_count !== 0 ) ||
                    (selected_item !== 'likeboards' && likeboard_count !== 0 ) ||
                    (selected_item !== 'brands' && brand_count !== 0 )
                ">
                    <span class="or"></span>
                    <p class="text">See the results in these categories</p>
                    <Button v-if="selected_item !=='people'  && people_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/people?query=' + search_keyword_encode">People</Button>
                    <Button v-if="selected_item !=='posts'  && post_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/posts?query=' + search_keyword_encode">Posts</Button>
                    <Button v-if="selected_item !=='products'  && product_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/products?query=' + search_keyword_encode">Products</Button>
                    <Button v-if="selected_item !=='hashtags'  && hashtag_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/hashtags?query=' + search_keyword_encode">#hashtag</Button>
                    <Button v-if="selected_item !=='likeboards'  && likeboard_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/likeboards?query=' + search_keyword_encode">LikeBoards</Button>
                    <Button v-if="selected_item !=='brands'  && brand_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/brands?query=' + search_keyword_encode">Brands</Button>
                </div>
            </div>
            <div class="top-list" v-else-if="selected_item === 'top'">
                <div v-if="people_count !== 0" class="top-list-item">
                    <h2 class="heading">People</h2>
                    <UserItem v-for="(item, index) in people_list" v-if="index < 3" :key="item.id +'-'+ index" :item="item" />
                    <div class="more-link" v-if="people_count > 3"><router-link :to="'/globalsearch/people?query=' + search_keyword_encode">See all {{people_count}} results</router-link></div>
                </div>
                <div v-if="hashtag_count !== 0" class="search-hashtag-list top-list-item">
                    <h2 class="heading">Hashtags</h2>
                    <ul>
                        <li v-for="(h, index) in hashtag_list" v-if="index < 3" class="selected" :key="h.id" @click="$router.push(`/trending/product/${h.resource_type === 'App\\TikTokVideo' ? 'tt' : 'ig'}/${h.text}`)">
                            <div class="icon">
                                <svg><use :xlink:href="'#hashtag-color--sprite'"></use></svg>
                            </div>
                            <Icon v-if="h.resource_type === 'App\\TikTokVideo'" :type="'tikTok'" :icon="'tiktok-no-circle'" :size="'extra-small'" :shape="'circle'" />
                            <Icon v-if="h.resource_type === 'App\\InstagramMedia'" :type="'instagram'" :icon="'instagram-no-circle'" :size="'extra-small'" :shape="'circle'" />
                            <div class="text">#{{h.text}}</div>
                            <div class="btn"><svg class="arrow2"><use xlink:href="#back--sprite"></use></svg></div>
                        </li>
                    </ul>
                    <div class="more-link" v-if="hashtag_count > 3"><router-link :to="'/globalsearch/hashtags?query=' + search_keyword_encode">See all {{hashtag_count}} results</router-link></div>
                </div>
                <div v-if="product_count !== 0" class="top-list-item top-list-item--products">
                    <h2 class="heading">Products</h2>
                    <Product v-for="(product, index) in product_list" image_size="400x400"  v-if="index < 4" :product="product" :key="product.id +'-'+ index" :show_info="true" :btn_type="'Like'"/>

                    <div class="more-link" v-if="product_count > 4"><router-link :to="'/globalsearch/products?query=' + search_keyword_encode">See all {{product_count}} results</router-link></div>
                </div>
                <div v-if="post_count !== 0" class="top-list-item">
                    <h2 class="heading">Posts</h2>
                    <Post v-for="(post, index) in post_list" v-if="index < 2" :key="index" v-bind:post="post" />

                    <div class="more-link" v-if="post_count > 4"><router-link :to="'/globalsearch/posts?query=' + search_keyword_encode">See all {{post_count}} results</router-link></div>
                </div>
                <div v-if="brand_count !== 0" class="top-list-item">
                    <h2 class="heading">Brands</h2>
                    <UserItem v-for="(item, index) in brand_list" v-if="index < 4" :key="item.id +'-'+ index" :item="item" />

                    <div class="more-link" v-if="brand_count > 4"><router-link :to="'/globalsearch/brands?query=' + search_keyword_encode">See all {{brand_count}} results</router-link></div>
                </div>
                <div v-if="likeboard_count !== 0" class="top-list-item top-list-item--likeboards">
                    <h2 class="heading">Likeboards</h2>
                    <Item v-for="(item, index) in likeboard_list" v-if="index < 4" :key="item.id" :item="item" :layout="'small'" :info_type="2" :border_color="true" />

                    <div class="more-link" v-if="likeboard_count > 4"><router-link :to="'/globalsearch/likeboards?query=' + search_keyword_encode">See all {{likeboard_count}} results</router-link></div>
                </div>
            </div>

            <div class="list" v-else-if="selected_item === 'posts' && post_count !== 0 ">
                <Post v-for="(post, index) in post_list" :key="index" v-bind:post="post" />
            </div>

            <div class="top-list-item" v-else-if="selected_item === 'people' && people_count !== 0">
                <UserItem v-for="(item, index) in people_list" :key="item.id +'-'+ index" :item="item" />
            </div>

            <div class="search-product-list" v-else-if="selected_item === 'products' && product_count !==0">
                <Product v-for="(product, index) in product_list" image_size="400x400" :product="product" :key="product.id +'-'+ index" :show_info="true" :btn_type="'Like'"/>
            </div>

            <div style="padding: .6rem;" class="top-list-item search-hashtag-list" v-else-if="selected_item === 'hashtags' && hashtag_count !==0">
                <ul>
                    <li v-for="h in hashtag_list" class="selected" :key="h.id" @click="$router.push(`/trending/product/${h.resource_type === 'App\\TikTokVideo' ? 'tt' : 'ig'}/${h.text}`)">
                        <div class="icon">
                            <svg><use :xlink:href="'#hashtag-color--sprite'"></use></svg>
                        </div>
                        <Icon v-if="h.resource_type === 'App\\TikTokVideo'" :type="'tikTok'" :icon="'tiktok-no-circle'" :size="'extra-small'" :shape="'circle'" />
                        <Icon v-if="h.resource_type === 'App\\InstagramMedia'" :type="'instagram'" :icon="'instagram-no-circle'" :size="'extra-small'" :shape="'circle'" />
                        <div class="text">#{{h.text}}</div>
                        <div class="btn"><svg class="arrow2"><use xlink:href="#back--sprite"></use></svg></div>
                    </li>
                </ul>
            </div>

            <div class="likeboard-list " v-if="selected_item === 'likeboards' && likeboard_count !==0">
                <Item v-for="item in likeboard_list" :key="item.id" :item="item" :layout="'small'" :info_type="2" :border_color="true" />
            </div>

            <div class="top-list-item" v-if="selected_item === 'brands' && brand_count !== 0">
                <UserItem v-for="(item, index) in brand_list" :key="item.id +'-'+ index" :item="item" />
            </div>

            <div class="zero-results" style="padding-top: 2rem;" v-if="
                (selected_item === 'products' && product_count !== 0 && is_end.products ) ||
                (selected_item === 'posts' && post_count !== 0 && is_end.posts ) ||
                (selected_item === 'people' && people_count !== 0 && is_end.people ) ||
                (selected_item === 'hashtags' && hashtag_count !== 0 && is_end.hashtags ) ||
                (selected_item === 'likeboards' && likeboard_count !== 0 && is_end.likeboards ) ||
                (selected_item === 'brands' && brand_count !== 0 && is_end.brand )
            ">
                <div class="btns" v-if="
                    (selected_item !== 'products' && product_count !== 0 ) ||
                    (selected_item !== 'posts' && post_count !== 0 ) ||
                    (selected_item !== 'people' && people_count !== 0 ) ||
                    (selected_item !== 'hashtags' && hashtag_count !== 0 ) ||
                    (selected_item !== 'likeboards' && likeboard_count !== 0 ) ||
                    (selected_item !== 'brands' && brand_count !== 0 )
                ">
                    <p class="text">See the results in these categories</p>
                    <Button v-if="selected_item !=='people'  && people_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/people?query=' + search_keyword_encode">People</Button>
                    <Button v-if="selected_item !=='posts'  && post_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/posts?query=' + search_keyword_encode">Posts</Button>
                    <Button v-if="selected_item !=='products'  && product_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/products?query=' + search_keyword_encode">Products</Button>
                    <Button v-if="selected_item !=='hashtags'  && hashtag_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/hashtags?query=' + search_keyword_encode">#hashtag</Button>
                    <Button v-if="selected_item !=='likeboards'  && likeboard_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/likeboards?query=' + search_keyword_encode">LikeBoards</Button>
                    <Button v-if="selected_item !=='brands'  && brand_count !== 0" :type="'bg'" :color="'orange'" :size="'32'" :to="'/globalsearch/brands?query=' + search_keyword_encode">Brands</Button>
                </div>
            </div>
            
        </section>

        <Loading v-if="is_loading_posts || is_loading_next" :classname="['static', 'nobg']" :requestURL="requestURL"/>

        <ProductPopup />
    </div>
</template>

<script>
import "@/assets/hashtag-color.svg?sprite";
import Button from '@/components/ui/Button.vue';
import List from '@/components/ui/List.vue';
import UserItem from '@/components/ui/ListItem.vue';
// import Product from '@/components/product/SearchProductItem.vue';
import Post from '@/components/Post.vue';
import Loading from '@/components/Loading.vue';
import Item from '@/components/likeboards/LikeboardItem.vue';
import SearchBar from '@/components/globalsearch/Bar.vue';
import NotFound from '@/components/globalsearch/NotFound.vue';
import Navi from '@/components/Navi.vue';
import Icon from '@/components/ui/Icon.vue';
import Product from '@/components/product/ProfileProductItem.vue';
import ProductPopup from '@/components/product/ProfileProductItemPopup.vue';
import '@/assets/follow-person.svg?sprite';
import '@/assets/google-g-no-circle.svg?sprite';
import '@/assets/instagram-no-circle.svg?sprite';
import '@/assets/facebook-no-circle.svg?sprite';
import '@/assets/tiktok-no-circle.svg?sprite';
import '@/assets/youtube-no-circle.svg?sprite';
import '@/assets/twitter-no-circle.svg?sprite';
import '@/assets/forward-arrow.svg?sprite';

export default {
    name: 'globalsearch',

    watch: {
        '$route.params': {
            handler(params) {
                if (params.filter_type === 'posts') {
                    this.timer = window.setTimeout(() => {
                        window.dispatchEvent(new Event('resize'));
                    }, 1000);
                }
                this.selected_item = params.filter_type;
                this.refreshData(params);
            },
            immediate: true
        }
    },

    components: {
        Loading,
        Button,
        Post,
        Product,
        List,
        Item,
        SearchBar,
        UserItem,
        Navi,
        NotFound,
        ProductPopup,
        Icon,
    },

    data() {
        return {
            is_end: {
                people: false,
                brand: false,
                posts: false,
                products: false,
                hashtags: false,
                likeboards: false,
            },
            menus: [],
            people_count: 0,
            product_count: 0,
            post_count: 0,
            hashtag_count: 0,
            brand_count: 0,
            likeboard_count: 0,
            is_loading_posts: false,
            is_loading_next: false,
            search_keyword: '',
            search_keyword_encode: '',
            selected_item: 'top',
            ajax_count: 0,
            people_list: [],
            product_list: [],
            post_list: [],
            hashtag_list: [],
            brand_list: [],
            likeboard_list: [],
            post_cursor: '',
            likeboard_cursor: '',
            product_cursor: '',
            people_cursor: '',
            brand_next_page: '',
            hashtag_next_page: '',
            requestURL: '',
        };
    },

    methods: {
        // searchUrl(type) {
        //     return `/globalsearch/${type}?query=${this.search_keyword_encode}`;
        // },

        refreshData(params) {
            const self = this;
            const keyword = decodeURI(self.$route.query.query);

            this.search_keyword = keyword;
            this.search_keyword_encode = encodeURIComponent(this.$route.query.query);

            if (this.$route.params.filter_type) {
                this.selected_item = this.$route.params.filter_type;
            } else {
                this.selected_item = 'posts';
            }

            this.people_list = [],
            this.product_list = [],
            this.post_list = [],
            this.brand_list = [],
            this.likeboard_list = [],

            this.saveGlobalSearch();
            this.searchPosts();
            this.searchProducts();
            this.searchLikeboards();
            this.searchPeople();
            this.searchBrands();
            this.searchHashtags();

            this.menus = [
                {
                    text: 'Top',
                    link: '/globalsearch/top?query=' + this.search_keyword_encode
                },
                {
                    text: 'People',
                    link: '/globalsearch/people?query=' + this.search_keyword_encode
                },
                {
                    text: 'Hashtags',
                    link: '/globalsearch/hashtags?query=' + this.search_keyword_encode
                },
                {
                    text: 'Products',
                    link: '/globalsearch/products?query=' + this.search_keyword_encode
                },
                {
                    text: 'Posts',
                    link: '/globalsearch/posts?query=' + this.search_keyword_encode
                },
                {
                    text: 'Brands',
                    link: '/globalsearch/brands?query=' + this.search_keyword_encode
                },
                {
                    text: 'LikeBoards',
                    link: '/globalsearch/likeboards?query=' + this.search_keyword_encode
                },
            ];
        },

        searchPeople() {
            const self = this;
            if (typeof self.search_keyword !== 'undefined' && self.search_keyword !== '') {
                self.is_loading_posts = true;
                self.requestURL = '/search/people/' + encodeURIComponent(self.search_keyword);
                self.$ajax('/search/people/' + encodeURIComponent(self.search_keyword), {
                    method: 'get',
                    withCredentials: true
                }).then(res => {
                    if (res.data.status === 'success') {
                        const list = res.data.data.data;
                        self.people_count = res.data.data.found;
                        self.people_cursor = res.data.data.cursor;
                        self.people_list = list;
                        self.is_end.people = res.data.data.cursor === null;
                    }

                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                }).catch(err => {
                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                    console.log(err);
                });
            }
        },

        searchProducts() {
            const self = this;
            if (typeof self.search_keyword !== 'undefined' && self.search_keyword !== '') {
                self.is_loading_posts = true;
                self.requestURL = '/search/product/' + encodeURIComponent(self.search_keyword);
                self.$ajax('/search/product/' + encodeURIComponent(self.search_keyword), {
                    method: 'get'
                }).then(res => {


                    if (res.data.status === 'success') {
                        self.is_end.products = res.data.data.cursor === null;
                        self.product_list = res.data.data.data;
                        self.product_count = res.data.data.found;
                        // self.product_cursor = self.product_list.length < 50 ? '' : (res.data.data.page + 1);
                        self.product_cursor = res.data.data.cursor;
                    }

                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                }).catch(err => {
                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                });
            }
        },

        searchPosts() {
            const self = this;
            if (typeof self.search_keyword !== 'undefined' && self.search_keyword !== '') {
                self.is_loading_posts = true;
                self.requestURL = '/search/post/' + encodeURIComponent(self.search_keyword);
                self.$ajax('/search/post/' + encodeURIComponent(self.search_keyword), {
                    method: 'get'
                }).then(res => {


                    if (res.data.status === 'success') {
                        self.post_list = res.data.data.data;
                        self.post_count = res.data.data.found;
                        self.post_cursor = res.data.data.cursor;
                        self.is_end.posts = res.data.data.cursor === null;
                    }

                    self.ajax_count ++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                }).catch(err => {
                    self.ajax_count ++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                });
            }
        },

        searchBrands() {
            const self = this;
            if (typeof self.search_keyword !== 'undefined' && self.search_keyword !== '') {
                self.is_loading_posts = true;
                self.requestURL = '/search/brand/' + encodeURIComponent(self.search_keyword);
                self.$ajax('/search/brand/' + encodeURIComponent(self.search_keyword), {
                    method: 'get'
                }).then(res => {
                    if (res.data.status === 'success') {
                        self.brand_list = res.data.data.data;
                        self.brand_count = res.data.data.total;
                        self.brand_next_page = res.data.data.next_page_url;
                        self.is_end.brand = self.brand_list.length < 20;
                    }

                    self.ajax_count++;

                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                }).catch(err => {
                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                });
            }
        },

        searchHashtags() {
            const self = this;
            if (typeof self.search_keyword !== 'undefined' && self.search_keyword !== '') {
                self.is_loading_posts = true;
                self.requestURL = '/search/hashtag/' + encodeURIComponent(self.search_keyword);
                self.$ajax('/search/hashtag/' + encodeURIComponent(self.search_keyword), {
                    method: 'get'
                }).then(res => {
                    if (res.data.status === 'success') {
                        self.hashtag_list = res.data.data.data;
                        self.hashtag_count = res.data.data.total;
                        self.hashtag_next_page = res.data.data.next_page_url;
                        self.is_end.hashtags = self.hashtag_list.length < 20;
                    }

                    self.ajax_count++;

                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                }).catch(err => {
                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                });
            }
        },

        searchLikeboards() {
            const self = this;
            if (typeof self.search_keyword !== 'undefined' && self.search_keyword !== '') {
                self.is_loading_posts = true;
                self.requestURL = '/search/likeboard/' + encodeURIComponent(self.search_keyword);
                self.$ajax('/search/likeboard/' + encodeURIComponent(self.search_keyword), {
                    method: 'get'
                }).then(res => {



                    if (res.data.status === 'success') {
                        const list = res.data.data.data;
                        self.likeboard_list = list;
                        self.likeboard_count = res.data.data.found;
                        self.likeboard_cursor = res.data.data.cursor;
                    }

                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }

                    self.is_end.likeboards = true;
                }).catch(err => {
                    self.ajax_count++;
                    if (self.ajax_count === 6) {
                        self.is_loading_posts = false;
                        self.ajax_count = 0;
                    }
                });
            }
        },

        getNextPage(cursor) {
            const self = this;
            let api = '';

            switch (self.selected_item) {
                case 'people':
                    api = '/search/people/' + encodeURIComponent(self.search_keyword) + '?cursor=' + cursor;
                    break;

                case 'products':
                    api = '/search/product/' + encodeURIComponent(self.search_keyword) + '?cursor=' + cursor;
                    break;

                case 'posts':
                    api = '/search/post/' + encodeURIComponent(self.search_keyword) + '?cursor=' + cursor;
                    break;

                case 'likeboards':
                    api = '/search/likeboard/' + encodeURIComponent(self.search_keyword) + '?cursor=' + cursor;
                    break;

                case 'brands':
                    api = self.brand_next_page;
                    break;

                case 'hashtags':
                    api = self.hashtag_next_page;
                    break;
            }

            self.requestURL = api;
            self.$ajax(api, {
                method: 'get',
                withCredentials: true
            }).then(res => {
                if (res.data.status === 'success') {
                    const list = res.data.data.data;

                    switch (self.selected_item) {
                    case 'people':
                        self.people_cursor = res.data.data.cursor;
                        break;

                    case 'products':
                        self.product_cursor = res.data.data.cursor;
                        // self.product_cursor = (res.data.data.page + 1);
                        // self.is_end.products = res.data.data.data.length < 50;
                        break;

                    case 'posts':
                        self.post_cursor = res.data.data.cursor;
                        break;

                    case 'likeboards':
                        self.likeboard_cursor = res.data.data.cursor;
                        break;

                    case 'brands':
                        self.brand_next_page = res.data.data.next_page_url;
                        break;

                    case 'hashtags':
                        self.hashtag_next_page = res.data.data.next_page_url;
                        break;
                    }

                    // self.people_count = res.data.data.found;
                    // self.people_cursor = res.data.data.cursor;

                    list.forEach(item => {
                        switch (self.selected_item) {
                        case 'people':

                            if (typeof item.post_author_social_accounts !== 'undefined' && item.post_author_social_accounts !== null && item.post_author_social_accounts.length !== 0) {
                                item.post_author_social_accounts.forEach(item => {
                                    item.icon = item.social_platform;
                                    switch (item.social_platform) {
                                    case 'google':
                                        item.link = 'https://myaccount.google.com/';
                                        item.icon = 'google-g-no-circle';
                                        break;
                                    case 'facebook':
                                        item.link = item.payload.link;
                                        item.icon = 'facebook-no-circle';
                                        break;
                                    case 'instagram':
                                        item.link = 'https://www.instagram.com/' + item.payload.username;
                                        item.icon = 'instagram-no-circle';
                                        break;
                                    case 'youtube':
                                        item.link = 'https://www.youtube.com/channel/' + item.payload.id;
                                        item.icon = 'youtube-no-circle';
                                        break;
                                    case 'twitter':
                                        item.link = 'https://www.twitter.com/' + item.payload.nickname;
                                        item.icon = 'twitter-no-circle';
                                        break;
                                    case 'tiktok':
                                        item.link = 'https://www.tiktok.com/' + item.payload.nickname;
                                        item.icon = 'tiktok-no-circle';
                                        break;
                                    }
                                });
                            }

                            self.people_list.push(item);
                            break;

                        case 'products':
                            self.product_list.push(item);
                            break;

                        case 'posts':
                            self.post_list.push(item);
                            break;

                        case 'brands':
                            self.brand_list.push(item);
                            break;

                        case 'likeboards':
                            self.likeboard_list.push(item);
                            break;

                        case 'hashtags':
                            self.hashtag_list.push(item);
                            break;
                        }
                    });
                }

                self.is_loading_next = false;
            }).catch(err => {
                self.is_loading_next = false;
                console.log(err);
            });
        },

        handleScroll() {
            const self = this;
            const body = document.body;
            const html = document.documentElement;
            const docHeight = Math.max(body.scrollHeight, body.offsetHeight, html.clientHeight, html.scrollHeight, html.offsetHeight);
            const scrollY = Math.max(0, html.scrollTop || window.pageYOffset || 0);
            let cursor = '';
            let is_next = false;

            switch (self.selected_item) {
                case 'people':
                    is_next = self.people_cursor !== null;
                    cursor = self.people_cursor;
                    this.is_end.people = !is_next;
                    break;

                case 'products':
                    // is_next = (self.product_cursor !== '') ? self.product_count > self.product_list.length : false;
                    // cursor = self.product_cursor;
                    // self.is_end.products = !is_next;
                    is_next = (self.product_cursor !== null);
                    cursor = self.product_cursor;
                    self.is_end.products = !is_next;
                    break;

                case 'posts':
                    is_next = (self.post_cursor !== null);
                    cursor = self.post_cursor;
                    self.is_end.posts = !is_next;
                    break;

                case 'likeboards':
                    is_next = (self.likeboard_cursor !== null);
                    cursor = self.likeboard_cursor;
                    self.is_end.likeboards = !is_next;
                    break;

                case 'brands':
                    is_next = (self.brand_next_page !== null);
                    cursor = true;
                    this.is_end.brand = !is_next;
                    break;

                case 'hashtags':
                    is_next = (self.hashtag_next_page !== null);
                    cursor = true;
                    this.is_end.hashtags = !is_next;
                    break;
            }


            if (
                typeof self.getNextPage === 'function' &&
                is_next &&
                !self.is_loading_posts &&
                !self.is_loading_next &&
                cursor !== '' &&
                scrollY > (docHeight - (window.innerHeight * 1.7))
            ) {
                self.is_loading_next = true;
                self.getNextPage(cursor);
            }
        }

    },

    created () {
        window.addEventListener('scroll', this.handleScroll);
    },

    destroyed () {
        window.removeEventListener('scroll', this.handleScroll);
    }
};

</script>
<style lang="scss">

    #mobile-wrap.page--globalsearch {
        background-color:#FDFAF8;
        min-height: 100vh;
        min-height: calc(var(--vh, 1vh) * 100);

        .header {
            display: none;
        }

        .search-header {
            position: sticky;
            box-shadow: 0 0 20px rgba(71, 71, 71, .1);
            background: white;
            top: 0;
            z-index: 999;

            .global-search-bar {
                position: relative;

                .box {
                    transform: unset;
                    position: relative;
                    border-radius: 0;
                }
                
                .bg {
                    display: none;
                }
            }

            .site-navbar {
                width: auto;
                justify-content: center !important;
                
                nav {
                    width: 100% !important;
                }
            }
        }
        
        .total-results {
            margin: 1.6rem;
        }

        .search-product-list {
            display: flex;
            flex-wrap: wrap;
        }

        .search-hashtag-list {
            ul {
                    position: relative;

                li {
                    width: 100%;
                    display: flex;
                    padding: 1rem;
                    cursor: pointer;

                    &:hover {
                        background-color: #FFF7E9;
                    }

                    .number {
                        width: 2rem;
                        font-weight: 500;
                        font-size: 14px;
                        line-height: 18px;
                        color: #58595B;
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        margin-right: 0.4rem;
                    }

                    .icon {
                        width: 32px;
                        height: 32px;
                        border: 1px solid #F2F2F2;
                        border-radius: 100px;
                        display: flex;
                        justify-content: center;
                        align-items: center;

                        svg {
                            width: 1.2rem;
                            height: 1.4rem;
                        }
                    }

                    .ui-icon {
                        position: relative;
                        left: -0.6rem;
                        top: 0.8rem;

                        &.shape-circle.size-extra-small {
                            width: unset !important;
                            height: unset !important;
                        }
                    }

                    .text {
                        flex: 1;
                        font-weight: 600;
                        font-size: 14px;
                        line-height: 18px;
                        color: #58595B;
                        display: flex;
                        align-items: center;
                        margin-left: 0.8rem;
                    }

                    .btn {
                        max-width: 8rem;
                        display: flex;
                        align-items: center;
                        margin: auto 0;
                        .arrow2,
                        .arrow {
                            position: relative;
                            height: 1.2rem;
                            width: .8rem;
                            fill: #828282;
                            transform: rotate(-180deg);
                            top: 50%;
                            margin: 0.4rem;
                        }
                    }
                }
            
            }
        }

        .container {
            height: auto;
            min-height: unset;
        }


        .list {
            padding: 0 1.2rem;
            overflow: hidden;
        }


        .search-content {
            margin: 0 auto;
            max-width: var(--modal-max-width);
            margin-left: auto;
            margin-right: auto;

            .likeboard-list {
                display: flex;
                flex-wrap: wrap;
            }

            .likeboard-list .likeboard-item {
                width: calc(50% - .4rem);
                box-sizing: border-box;
                margin-bottom: .8rem;

                &:nth-child(odd) {
                    margin-right: .8rem;
                }
            }
        }


        .search-product-list {
            .product-item {
                width: calc(50% - .4rem);
                margin-bottom: .8rem;

                &:nth-child(odd) {
                    margin-right: .8rem;
                }
            }
        }

        @at-root {
            @media (max-width: 780px) {
                & {
                    .search-product-list,
                    .likeboard-list {
                        padding: 0 1.6rem;
                    }
                }
            }
        }
    }

    .top-list-item .ui-list-item {
        margin: .8rem 0 0;


        &.type-brand .inner, 
        &.type-user .inner {
            padding: 0;
        }

        &::before {
            display: none
        }

        .info .about-me {
            display: none;
        }
    }

</style>
<style scoped lang="scss">
    @import '../resources/sass/_variables.scss';

    .top-list-item {
        margin: .8rem 0;
        background: white;
        padding: 1.6rem;

        .heading {
            font-size: 1.6rem;
            color: var(--black-90);
            font-weight: 600;
            margin-bottom: 1.6rem;
            width: 100%;
        }
        .more-link {
            text-align: center;
            margin: 1.6rem 0 0;
            width: 100%;

            a {
                text-align: center;
                color: var(--cl-primary);
                font-weight: 600;
            }
        }

        &.search-hashtag-list {
            padding-left: .6rem;
            padding-right: .6rem;

            .heading {
                padding: 0 1rem;
            }
        }

        &--products {
            display: flex;
            flex-wrap: wrap;

            .product-item {
                width: calc(50% - .4rem);
                margin-bottom: .8rem;

                &:nth-child(odd) {
                    margin-left: .8rem;
                }
            }
        }

        &--likeboards {
            display: flex;
            flex-wrap: wrap;

            .likeboard-item {
                width: calc(50% - .4rem);
                margin-bottom: .8rem;

                &:nth-child(odd) {
                    margin-left: .8rem;
                }
            }
        }
    }
 
    .end-text {
        text-align:center;
        font-weight: 500;
        font-size: 1.6rem;
        color: $med_grey;
        padding: .8rem 0 2.6rem;
    }

    .small-text {
        font-size: 1.2rem;
        color: $med_grey;
        font-weight: 500;
    }

    .spinner-text {
        padding: 1.6rem .4rem .8rem;
        width: 100%;
        text-align: center;
        .spinner-description {
            font-weight: 600;
        }
    }

    .result-text {
        font-weight: 600;

        strong {
            color: $orange;
            font-size: 2rem;
        }
    }

    .zero-results {
        padding-top: 8vh;
        text-align: center;
    }

    .btns {
        margin: 0 auto 5rem auto;
        max-width: 34rem;
        font-weight: 500;

        .or {
            display: block;
            margin: 2rem 0 2rem;
            position: relative;
            z-index: 1;

            &::after {
                content: '';
                position: absolute;
                height: 1px;
                width: 100%;
                left: 0;
                top: 50%;
                background: var(--black-20);
            }

            &::before {
                content: 'or';
                background: var(--bg-05);
                position: relative;
                z-index: 3;
                color: var(--black-75);
                padding: 0 1rem;
            }
        }

        .text {
            margin: 0 0 1rem;
            color: var(--black-75);
        }

        .ui-button {
            margin: .5rem .4rem;
        }
    }

</style>
