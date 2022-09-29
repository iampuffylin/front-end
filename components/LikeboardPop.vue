<template>
    <div>
        <div class="likeboard-info" ref="share">
            <div class="likeboard-info--wrapper">
                <div class="likeboard-info--container">
                    <header class="likeboard-info--header">
                        <h2 class="likeboard-info--title">{{ info_type === 0 ? 'Create New LikeBoard' : 'Edit LikeBoard info' }}</h2>
                        <div class="likeboard-info--close" @click="close(true)">
                            <svg><use xlink:href="#close--sprite" /></svg>
                        </div>
                    </header>

                    <div class="likeboard-info--body">
                        <div class="box">
                            <h5 class="required">LikeBoard Name</h5>
                            <input v-model="name" @keyup="countChar('name')" type="text" maxlength="60" placeholder="LikeBoard Name" />
                            <p>{{ name_count }}/60</p>
                        </div>
                        <div class="box">
                            <h5>Describe your LikeBoard</h5>
                            <textarea v-model="description" @keyup="countChar('desc')" maxlength="99" placeholder="Optional description"></textarea>
                            <p>{{ desc_count }}/99</p>
                        </div>
                        <div class="box">
                            <h5>Color theme</h5>
                            <div class="colors">
                                <ul>
                                    <li v-for="index in 14" :key="index" @click="changeColor(index-1)">
                                            <div :class="['border', 'color-'+(index-1), {'actived': index-1 === color_id}]"><div :class="['circle', 'color-'+(index-1), {'actived': index-1 === color_id}]" /></div>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <div class="box">
                            <div class="is-public">
                                <h5>Set to public</h5>
                                <!-- Rounded switch -->
                                <div>
                                    <label class="switch" >
                                    <input type="checkbox" @click="is_public = !is_public">
                                    <span class="slider round"></span>
                                    </label>
                                </div>
                            </div>
                        </div>
                    </div>

                    <footer class="likeboard-info--footer">
                        <div class="apply-button" v-if="show_next" @click="next" :class="{'actived': name !== ''}">Next</div>
                        <div class="apply-button" v-else @click="apply" :class="{'actived': name !== ''}">{{ info_type === 0 ? 'Create' : 'Save' }}</div>
                    </footer>
                </div>
                <div class="likeboard-info--bg" @click="close(false)"></div>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'LikeboardInfoPopUp',

    components: {
    },

    data() {
        return {
            info_type: 0,
            name: '',
            description: '',
            name_count: 0,
            desc_count: 0,
            color_id: 0,
            is_public: false,
        };
    },

    props: {
        likeboard: Object,
        show_next: {
            type: Boolean,
            default: false,
        },
    },
    
    watch: {
        show_likeboards: {
            handler: function() {
                if (this.show_likeboards) {
                    this.show_title = 'Add/Remove to';
                } else {
                    this.show_title = 'What would you like to do?';
                }
            }
        }
    },

    methods: {

        open(info_type) {
            this.info_type = info_type;

            this.$refs.share.classList.add('open');

            setTimeout(() => {
                this.$refs.share.classList.add('open-bg');
            }, 10);

            setTimeout(() => {
                this.$refs.share.classList.add('open-container');
            }, 150);
        },

        close(is_click_button_close) {
            const self = this;

            if (self.$refs.share) self.$refs.share.classList.remove('open-container');

            setTimeout(() => {
                if (self.$refs.share) self.$refs.share.classList.remove('open-bg');
            }, 200);

            setTimeout(() => {
                if (self.$refs.share) self.$refs.share.classList.remove('open');
                self.$emit('closeCallback');

                if (is_click_button_close) {
                    self.$emit('callback');
                }
            }, 300);
        },

        changeColor(inx) {
            this.color_id = inx;
        },

        countChar(type) {
            if (type === 'name')
                this.name_count = this.name.length;
            else
                this.desc_count = this.description.length;
        },

        apply() {
            if (this.name === '') return;
            this.close();
            if (this.info_type === 0) {
                let form_data = new FormData();
                form_data.append('name', this.name);
                form_data.append('description', this.description);
                form_data.append('is_public', (this.is_public) ? 1 : 0);
                form_data.append('is_default', 0);
                form_data.append('color', 'v2');
                form_data.append('color_id', this.color_id);

                this.$ajax('/api/likeboards', {
                    method: 'post',
                    data: form_data,
                    withCredentials: true,
                }).then(res => {
                    res.data.data.layout = '4-6';
                    res.data.data.is_public = Number(res.data.data.is_public);
                    res.data.data.created_default = 0;
                    // Removed all cache
                    this.$ajax.cache.store = {};

                    // self.likeboards.unshift(res.data.data);
                    // self.$emit('created', res.data.data);
                    this.close();
                    this.$emit('createLikeboard', res.data.data);

                }).catch(err => {
                    alert('The name has already been taken.');
                });
            }
        },

        next() {
            if (this.name === '') return;
            this.close();
            if (this.info_type === 0) {
                let form_data = new FormData();
                form_data.append('name', this.name);
                form_data.append('description', this.description);
                form_data.append('is_public', (this.is_public) ? 1 : 0);
                form_data.append('is_default', 0);
                form_data.append('color', 'v2');
                form_data.append('color_id', this.color_id);

                this.$ajax('/api/likeboards', {
                    method: 'post',
                    data: form_data,
                    withCredentials: true,
                }).then(res => {
                    res.data.data.layout = '4-6';
                    res.data.data.is_public = Number(res.data.data.is_public);
                    res.data.data.created_default = 0;
                    // Removed all cache
                    this.$ajax.cache.store = {};

                    this.$router.push('/me/likeboards/' + res.data.data.id + '/add-product');

                }).catch(err => {
                    alert('The name has already been taken.');
                });
            }
        },
    },
};
</script>

<style lang="scss">
    @import '../resources/sass/_variables.scss';

    .likeboard-info {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index:9999999;
        display: none;

        &--wrapper {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
        }

        &--container {
            position: absolute;
            z-index: 9;
            overflow: hidden;
            left: 50%;
            width: 100%;
            max-width: var(--overlay-max-width);
            box-shadow: var(--pop-medium);
            bottom: 0;
            transition: transform .3s ease;
            transform: translate3d(-50%, 100%, 0);
            background: white;
            border-radius:  1.6rem 1.6rem 0 0;
        }

        &--bg {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
            background: rgba(#000, .8);
            opacity: 0;
            transition: opacity .2s ease;
        }

        &--header {
            padding: 0 1.6rem;
            min-height: 4.8rem;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }

        &--title {
            font-size: 2rem;
            font-weight: 600;
            position: relative;
            background: white;
            color: $dark_grey;
            overflow: hidden;
        }

        &--close {
            height: 3.2rem;
            width: 3.2rem;
            color: white;
            cursor: pointer;
            position: relative;
            right: -.8rem;

            svg {
              height: 3.2rem;
              width: 3.2rem;
              display: block;
              fill: var(--black-50);
            }

            &:hover svg {
              fill: var(--cl-primary);
            }
        }

        &--body {
            color: $dark_grey;
            background: var(--bg-06);
            max-height: 70vh;
            overflow: auto;
            padding: 1.6rem;

            .box {
                margin: unset;
                padding: unset;
                border-radius: 12px;
                background-color: unset;
                margin-bottom: 1.6rem;

                // &:last-child {
                //     margin-bottom: 1.6rem;
                // }

                h5 {
                    margin-bottom: 0.6rem;
                    font-weight: 600;
                    font-size: 16px;
                    color: #333333;

                    &.required::after {
                        content: ' *';
                        color: var(--cl-primary);
                    }
                }

                input, textarea {
                    padding: 10px;
                    width: 100%;
                    background: #FFFFFF;
                    border: 1px solid #F2F2F2;
                    border-radius: 6px;
                    resize: none;
                }

                .colors {
                    margin: -.5rem 0 0;
                    width: 100%;
                    overflow: auto;

                    ul {
                        display: flex;
                    }

                    li {
                        $size: 4.4rem;
                        overflow: hidden;
                        width: $size;
                        height: $size;
                        min-width: $size;
                        cursor: pointer;

                        .border {
                            border-radius: 100px;
                            width: 4.2rem;
                            height: 4.2rem;
                            display: flex;
                            justify-content: center;
                            align-items: center;
                            border: 3px solid var(--bg-06);

                            &.color-0.actived {
                                border: 3px solid $lb_1;
                            }
                            &.color-1.actived {
                                border: 3px solid $lb_2;
                            }
                            &.color-2.actived {
                                border: 3px solid $lb_3;
                            }
                            &.color-3.actived {
                                border: 3px solid $lb_4;
                            }
                            &.color-4.actived {
                                border: 3px solid $lb_5;
                            }
                            &.color-5.actived {
                                border: 3px solid $lb_6;
                            }
                            &.color-6.actived {
                                border: 3px solid $lb_7;
                            }
                            &.color-7.actived {
                                border: 3px solid $lb_8;
                            }
                            &.color-8.actived {
                                border: 3px solid $lb_9;
                            }
                            &.color-9.actived {
                                border: 3px solid $lb_10;
                            }
                            &.color-10.actived {
                                border: 3px solid $lb_11;
                            }
                            &.color-11.actived {
                                border: 3px solid $lb_12;
                            }
                            &.color-12.actived {
                                border: 3px solid $lb_13;
                            }
                            &.color-13.actived {
                                border: 3px solid $lb_14;
                            }
                        }

                        .circle {
                            width: 3.12rem;
                            height: 3.12rem;
                            border-radius: 100px;
                            

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
                        }
                    }
                }

                p {
                    font-weight: 500;
                    font-size: 12px;
                    line-height: 14px;
                    color: #828282;
                    text-align: right;
                    margin-top: 0.6rem;
                }

                .is-public {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;

                    h5 {
                        margin-bottom: unset;
                    }

                    .switch {
                        position: relative;
                        display: inline-block;
                        width: 5rem;
                        height: 3.0rem;
                    }

                    /* Hide default HTML checkbox */
                    .switch input {
                        opacity: 0;
                        width: 0;
                        height: 0;
                    }

                    /* The slider */
                    .slider {
                        position: absolute;
                        cursor: pointer;
                        top: 0;
                        left: 0;
                        right: 0;
                        bottom: 0;
                        background-color: #ccc;
                        -webkit-transition: .4s;
                        transition: .4s;
                    }

                    .slider:before {
                        position: absolute;
                        content: "";
                        height: 2.2rem;
                        width: 2.2rem;
                        left: 0.4rem;
                        bottom: 0.4rem;
                        background-color: white;
                        -webkit-transition: .4s;
                        transition: .4s;
                    }

                    input:checked + .slider {
                        background-color:#F16529;
                    }

                    input:focus + .slider {
                        box-shadow: 0 0 1px #F16529;
                    }

                    input:checked + .slider:before {
                        transform: translateX(2rem);
                    }
                    /* Rounded sliders */
                    .slider.round {
                        border-radius: 3.4rem;
                    }

                    .slider.round:before {
                        border-radius: 50%;
                    }

                    .uncheck {
                        position: absolute;
                        opacity: 1;
                        height: 2rem;
                        width:  2rem;
                        margin: -.8rem 0 0;
                        top: 50%;
                        right: $padding;
                        fill: $black;
                    }
                }
            }

            
        }

        &--footer {
            padding: 1.6rem;
            border-top: 1px solid $light_grey;
            display: flex;

            .apply-button {
                @include btn-large-disabled;
                width: 100%;
                cursor: pointer;
                font-size: 16px;
                border-radius: 100px;
                letter-spacing: unset;
                cursor: default;

                &.actived {
                    @include btn-large-active;
                    border-radius: 100px;
                }
            }

            .reset-button {
                padding: 1rem;
                text-align: center;
                display: block;
                cursor: pointer;
                color: $orange;
                font-size: 16px;
                letter-spacing: unset;
            }
        }

        .ui-checkbox-item {
            $size: 2.4rem;
            background: white;
            padding: 1.2rem $padding $padding;
            display: flex;
            position: relative;

            label {
                flex: 1;
                color: $med_dk_grey;
                font-weight: 600;

                &::before {
                    position: absolute;
                    height:  100%;
                    width:  100%;
                    top: 0;
                    left: 0;
                    content: '';
                    z-index: 3;
                    cursor: pointer;
                }

                & > h4 {
                    margin: 0 0 .8rem;
                    text-align: left;
                    font-size: 1.4rem;
                }

                & > p {
                    font-size: 1rem;
                    text-align: left;
                }
            }

            input {
                position: absolute;
                left: 0;
                top: 0;
                opacity: 0;
                z-index: -1;
                visibility: hidden;
            }

            span {
                border: 1px solid $med_light_grey;
                box-sizing: border-box;
                height: $size;
                border-radius:  $size / 2;
                width:  $size * 1.8;
                display: block;
                position: relative;
                transition: all .2s;
                background: $grey_white;
                top: -.2rem;

                &::before {
                    position: absolute;
                    box-sizing: border-box;
                    box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.1);
                    border: 1px solid $med_light_grey;
                    height:  $size;
                    width:  $size;
                    top: -1px;
                    left: -1px;
                    content: '';
                    background: $white;
                    border-radius: 50%;
                    transition: all .2s;
                }
            }

            input:checked + span {

                &::before {
                    border: 1px solid $med_light_grey;
                    left: $size*.8;
                    background: $orange;

                }
            }
        }

        &.open {
            display: block;
        }

        &.open-container &--container {
            transform: translate3d(-50%, 0%, 0);
        }

        &.open-bg &--bg {
            opacity: 1;
        }
    }

    @media (min-width: $mobile_size) {
        .likeboard-info {

            &--container {
                opacity: 0;
                transition: opacity .2s ease;
                transform: translate3d(-50%, -50%, 0);
                border-radius:  1.6rem;
                right: auto !important;
                bottom: auto !important;
                left: 50%;
                top: 50%;
            }

            &--header {
                display: flex;
                align-items: center;
            }

            &.open-container &--container {
                transform: translate3d(-50%, -50%, 0);
            }

            &.open-bg &--container,
            &.open-bg &--bg {
                opacity: 1;
            }
        }
    }
</style>
<style lang="scss" scoped>
    
</style>