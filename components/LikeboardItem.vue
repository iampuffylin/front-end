<template>
<!-- eslint-disable -->

    <div class="likeboard-item" :class="['layout-' + layout, {post: is_featured}, {'border-color': border_color && info_type === 0}, 'color-' + item.color_id]" v-observe-visibility="{callback: sendPageView, once:true, intersection: {threshold: 0.5}, throttle: 1000}">
        
        <div class="box" v-if="item">
            <div :class="[{'border-color': border_color && info_type !== 0}, 'color-' + item.color_id]">
                <router-link :to="url">
                    <Layout :item="item" :is_edit="is_edit" />
                </router-link>
            </div>
            <div v-if="info_type === 1" class="info" :class="[{'is-private': !item.is_public}, {'old-version': item.color !== 'v2'}, {'new-version': item.color === 'v2'}, 'color-'+item.color_id]" :style="{'background-color': bg_color}">
                <h3><router-link :to="url">{{item.name}}</router-link></h3>
                <svg v-if="!item.is_public"><use xlink:href="#private2--sprite"></use></svg>
                <p v-if="is_own">
                    {{item.likes_count}} items
                </p>
                <p v-else-if="item.user_profile">
                    <router-link :to="{path: '/user/' + item.user_profile.user_id + '/profile'}">by {{item.user_profile.nickname}}</router-link>
                </p>
            </div>
            <div class="info-2" v-else-if="info_type === 2 || info_type === 3">
                <div class="left">
                    <div class="likeboard-name">{{item.name}}</div>
                    <div v-if="info_type === 2 && item.user_profile" class="user-name">
                        <router-link :to="{path: '/user/' + item.user_profile.user_id + '/profile'}">
                            <Avatar :size="'extra-small'" :image="item.user_profile.avatar" />
                            <span>{{item.user_profile.nickname}}</span>
                        </router-link>
                    </div>
                    <div v-else-if="info_type === 3" class="item-count">{{item.product_count}} items</div>
                </div>
                <div class="right">
                    <svg v-if="!item.is_public"><use xlink:href="#private2--sprite"></use></svg>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
/* eslint-disable */
import "@/assets/private2.svg?sprite";
import Layout from '@/components/likeboards/ImagesLayout.vue';
import Avatar from '@/components/ui/Avatar.vue';

export default {
    name: 'LikeboardItem',

    data() {
        return {
            url: '',
            is_own: false,
            bg_color: '',
        }
    },

    props: {
        item: Object,
        layout: String,
        is_featured: false,
        is_edit: {
            type: Boolean,
            default: false
        },
        border_color: {
            type: Boolean,
            default: false
        },
        info_type: {
            type: Number,
            default: 1
        },
    },


    components: {
        Layout,
        Avatar,
    },

    methods: {
        sendPageView(isVisible, entry) {
            let self = this;
            if (isVisible) {
                // go save pageview
                let likeBoardID = self.item.unique_key;
                if (likeBoardID) {
                    this.$ajax('/api/pageview/likeboard/' + likeBoardID + '?r=' + Math.random(), {
                        method: 'put',
                        withCredentials: true,
                    });
                }
            }            
        },
    },

    mounted() {


       	this.$nextTick(() => {
            if (this.item.is_public) {
                this.url = '/public-likeboards/' + this.item.unique_key;
            } else {
                this.url = '/likeboards/' + this.item.id;
            }

            if (this.item.color !== 'v2') this.bg_color = this.item.color;

            if (this.item.user_profile && Number(this.$store.state.user.id) === Number(this.item.user_profile.user_id)) {
                this.is_own = true;
            }
        });
    },

}
</script>

<style lang="scss">
    @import '../resources/sass/_variables.scss';

    .likeboard-item {

        &.border-color {
            padding: 0.4rem;
            border-radius: 0.8rem;

            &.color-0 {
                background: $lb_1;
            }

            &.color-1 {
                background: $lb_2;
            }

            &.color-2 {
                background: $lb_3;
            }

            &.color-3 {
                background: $lb_4;
            }

            &.color-4 {
                background: $lb_5;
            }

            &.color-5 {
                background: $lb_6;
            }

            &.color-6 {
                background: $lb_7;
            }

            &.color-7 {
                background: $lb_8;
            }

            &.color-8 {
                background: $lb_9;
            }

            &.color-9 {
                background: $lb_10;
            }

            &.color-10 {
                background: $lb_11;
            }

            &.color-11 {
                background: $lb_12;
            }

            &.color-12 {
                background: $lb_13;
            }

            &.color-13 {
                background: $lb_14;
            }

            &.layout-small {
                .likeboard-layout {
                    border-radius: 0.8rem !important;
                }
            }
            
            .likeboard-layout {
                border-radius: 0.8rem 0.8rem 0 0 !important;
            }

            .info-2 {
                border-radius: 0 0 0.8rem 0.8rem;
            }
        }

        .box {
            display: block;
            border-radius: .8rem;
            overflow: hidden;
            background: white;

            & > a {
                display: block;
                position: relative;
                background-color: $grey_white;
            
                &::after {
                    content: '';
                    display: block;
                    padding: 0 0 100%;
                }

                .likeboard-layout {
                    position: absolute;
                    left: 0;
                    top: 0;
                    height: 100%;
                    width: 100%;
                }
            
                .likeboard-layout::after {
                    display: none;
                }
            }

            .border-color {
                padding: 0.4rem;
                border-radius: 0.8rem;

                &.color-0 {
                    background: $lb_1;
                }

                &.color-1 {
                    background: $lb_2;
                }

                &.color-2 {
                    background: $lb_3;
                }

                &.color-3 {
                    background: $lb_4;
                }

                &.color-4 {
                    background: $lb_5;
                }

                &.color-5 {
                    background: $lb_6;
                }

                &.color-6 {
                    background: $lb_7;
                }

                &.color-7 {
                    background: $lb_8;
                }

                &.color-8 {
                    background: $lb_9;
                }

                &.color-9 {
                    background: $lb_10;
                }

                &.color-10 {
                    background: $lb_11;
                }

                &.color-11 {
                    background: $lb_12;
                }

                &.color-12 {
                    background: $lb_13;
                }

                &.color-13 {
                    background: $lb_14;
                }

                &.layout-small {
                    .likeboard-layout {
                        border-radius: 0.8rem !important;
                    }
                }
                
                .likeboard-layout {
                    border-radius: 0.8rem !important;
                }

                .info-2 {
                    background: white;
                    // margin-top: 0.4rem;
                    border-radius: 0 0 0.8rem 0.8rem;
                }
            }
        }

        .info {
            padding: .5rem 1rem;
            box-sizing: border-box;
            position: relative;
            color: white;
            min-height: 4.4rem;

            &.new-version {
                background: $lb_1;

                a {
                    color: white;
                }

                &.color-1 {
                    background: $lb_2;
                    color: $dark_grey;

                    a {
                        color: $dark_grey;
                    }
                }

                &.color-2 {
                    background: $lb_3;
                }

                &.color-3 {
                    background: $lb_4;
                }

                &.color-4 {
                    background: $lb_5;
                }

                &.color-5 {
                    background: $lb_6;
                }

                &.color-6 {
                    background: $lb_7;
                }

                &.color-7 {
                    background: $lb_8;
                    color: $dark_grey;

                    a {
                        color: $dark_grey;
                    }
                }

                &.color-8 {
                    background: $lb_9;
                }

                &.color-9 {
                    background: $lb_10;
                    color: $dark_grey;

                    a {
                        color: $dark_grey;
                    }
                }

                &.color-10 {
                    background: $lb_11;
                }

                &.color-11 {
                    background: $lb_12;
                }

                &.color-12 {
                    background: $lb_13;
                    color: $dark_grey;

                    a {
                        color: $dark_grey;
                    }
                }

                &.color-13 {
                    background: $lb_14;
                    color: $dark_grey;

                    a {
                        color: $dark_grey;
                    }
                }
            }
            

            h3 {
                font-weight: 600;
            }

            p {
                font-size: 1rem;
                
                a {
                    font-weight: 500;
                }
            }

            svg {
                position: absolute;
                height: 1.6rem;
                width: 1.6rem;
                right: 1rem;
                top: 50%;
                margin-top: -.8rem;
                fill: white;
            }

            &.is-private {
                padding-right: 4rem;
            }

            &.old-version {
                background-color: $light_grey;
                color: $dark_grey;

                svg {
                    fill: $dark_grey;
                }

                a {
                    color: $dark_grey;
                }
            }


        }

        .info-2 {
            display: flex;

            .left {
                display: grid;
                flex: 1;

                .likeboard-name {
                    font-weight: var(--semibold);
                    font-size: 1.2rem;
                    color: var(--black-90);
                    padding: 0.6rem;
                    overflow:hidden;
                    text-overflow:ellipsis;
                    white-space: nowrap;
                }

                .user-name {
                    margin: 0 0.6rem;
                    overflow: hidden;

                    .ui-avatar {
                        min-width: 1.6rem;
                        width: 1.6rem;
                        height: 1.6rem;
                        margin-right: 0.4rem;

                        div {
                            width: 1.6rem;
                            height: 1.6rem;
                            border: solid 0.1rem var(--black-20);
                        }
                    }

                    a {
                        font-weight: var(--medium);
                        font-size: 1rem;
                        color: var(--black-75);
                        display: flex;
                        align-items: center;
                        padding-bottom: 0.6rem;

                        span {
                            overflow:hidden;
                            text-overflow:ellipsis;
                            white-space: nowrap;
                        }
                    }
                }

                .item-count {
                    margin: 0 0.6rem 0.6rem 0.6rem;
                    font-weight: var(--medium);
                    font-size: 1rem;
                    color: var(--black-75);
                }
            }
            
            .right {
                display: flex;
                justify-content: center;
                align-items: center;

                svg {
                    height: 1.6rem;
                    width: 1.6rem;
                    fill: var(--black-50);
                    margin: 0 1.2rem;
                }
            }
        }
    }


</style>
