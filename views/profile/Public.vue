<template>
  <div class="container profile-container">
    <div class="container-inner">
      <Header
        v-if="user_data"
        :data="user_data"
      />
      <Loading
        v-else
        :classname="['static', 'nobg']"
      />

      <Menu v-if="user_data" :menus="menus" />
      <SocialDeck
        v-if="user_data && $route.name === 'public-profile-social'"
        :user_data="user_data"
        @follow="follow"
      />
      <Posts
        v-else-if="user_data && $route.name === 'public-profile'"
        :user_data="user_data"
        @follow="follow"
      />
      <StoreFront
        v-else-if="user_data && $route.name === 'public-profile-storefront'"
        :user_data="user_data"
      />
      <Tagged
        v-else-if="user_data && $route.name === 'public-profile-tagged'"
        :user_data="user_data"
      />
      <SeenIn
        v-else-if="user_data && $route.name === 'public-seen-in'"
        :user_data="user_data"
        @follow="follow"
      />
      <Likeboards
        v-else-if="user_data && ($route.name === 'public-likeboards' || $route.name === 'public-likeboards-like')"
        :user_data="user_data"
        @follow="follow"
      />
    </div>
  </div>
</template>

<script>
import '@/assets/social-deck.svg?sprite';
import '@/assets/shopbag.svg?sprite';

import Header from '@/components/profile/ProfileHeader.vue'
import Menu from '@/components/profile/Menu.vue'
import SocialDeck from '@/components/profile/SocialDeck.vue'
import Posts from '@/components/profile/Posts.vue'
import StoreFront from '@/components/profile/StoreFront.vue'
import Tagged from '@/components/profile/Tagged.vue'
import Likeboards from '@/components/profile/Likeboards.vue'
import SeenIn from '@/components/profile/SeenIn.vue'
import Loading from '@/components/Loading.vue'
import LoginFooter from '@/components/profile/LoginFooter.vue'

export default {
  name: 'PublicProfile',

  components: {
    Header,
    Menu,
    SocialDeck,
    Posts,
    StoreFront,
    Tagged,
    Likeboards,
    SeenIn,
    Loading,
    LoginFooter
  },

  data () {
    return {
      menus: [],
      meta_title: '',
      is_sticky: false,
      user_data: false,
      user_data_count: {},
      is_connected: false,
    }
  },

  watch: {
    '$route.params': {
      handler (params) {
        if (this.$route.params.handle) {
            this.$ajax('/api/user/handle-to-id', {
                method: 'post',
                data: { handle: this.$route.params.handle },
                withCredentials: true,
            }).then(res => {
                if (res && res.data && res.data.status === 'success') {
                    let user_id = res.data.data.user_id;
                    if (user_id) {
                        this.$route.params.user_id = user_id.toString();
                        this.refreshData();
                    }
                    else this.$router.push('/');
                }
            })
            .catch(err => {
                this.$router.push('/');
            });
        } else {
            this.$ajax('/api/user/id-to-handle', {
                method: 'post',
                data: { user_id: this.$route.params.user_id },
                withCredentials: true,
            }).then(res => {
                if (res && res.data && res.data.status === 'success') {
                    let handle = res.data.data.motom_handle;
                    let query = this.$route.query;
                    if (handle) {
                        let path = this.$route.path.substr(this.$route.path.indexOf('/',6));
                        let handle_url = '/@' + handle + path;
                        console.log(handle_url);
                        this.$router.replace( { path: handle_url, query: query } );
                    }
                    else this.refreshData();
                }
            })
            .catch(err => {
                this.refreshData();
            });
        }
      },
      immediate: true
    }
  },

  metaInfo() {

    return {
      title: this.meta_title,
      meta:[
        {
          name: 'description', 
          content: 'Buy trending outfits, styles & products from TikTok/Instagram influencers, trendsetters & fashion tastemakers. Motom creators don\'t gatekeep; shop their feeds!' 
        }
      ]
    }
  },

  methods: {
    refreshData () {
        if (this.isMotomApp) {
            this.$SmoothScroll(3, 500);
        } else {
            window.scrollBy(0, 0);
        }
        
        if (!this.$route.params.user_id) {
            window.location.replace('/')
            return
        }

        const user_id = this.$route.params.user_id

        // this.menus = [];
        // this.user_data = false;
        let pub_url = (this.$route.params.handle) ? `/@${this.$route.params.handle}` : `/user/${this.$route.params.user_id}`;

        Promise.all([
          this.$ajax.get('/api/user/' + user_id),
          this.$ajax.get('/api/user/' + user_id + '/post/check-have-post')
        ]).then(res => {
            this.menus = [];

            this.user_data = res[0].data;
        
            let user_name = (this.user_data.profiles.user_type && this.user_data.profiles.user_type === 2) ? 'Motom Creator Watch' : this.user_data.profiles.first_name  + ' ' + this.user_data.profiles.last_name;
            if (this.user_data.profiles.is_influencer === 1) {
              this.meta_title = `Styles From ${user_name} on Motom - Shop TikTok/Instagram Feeds`;
            } else {
              this.meta_title = `${user_name} on Motom - Shop TikTok/Instagram Fashion & Products`;
            }
            // this.meta_title = user_id;

            let social_deck = Object.assign({}, this.user_data.social_accounts);
            this.is_connected = social_deck.length === 0 || Object.keys(social_deck).length === 0 ? false : true;
            this.$store.commit('setCustomID', '00' + this.user_data.profiles.user.unique_key)
            this.menus.push({
                text: 'Posts',
                type: 'icon',
                icon: 'social-deck',
                link: `${pub_url}/profile`
            });

            this.menus.push({
              text: 'Shop',
              type: 'icon',
              icon: 'shopbag',
              link: `${pub_url}/storefront`
            });
        }).catch(() => {
            window.location.replace('/');
        })
    },

    follow () {
      this.$ajax('/api/post/user/' + this.user_data.user_id + '/follow', {
        method: 'post',
        withCredentials: true
      }).then(res => {
        if (res && res.data && res.data.status === 'success') {
          let label = window.user_id_alpha ? window.user_id_alpha : 'undefined'
          label = label + ',' + this.user_data.profiles.user.unique_key

          this.GA_Tracking('FOLLOW', 'CLICK', label)
          this.user_data.profiles.is_followed = true
        }
      }).catch(err => {
        if (err.response.data.message === 'Unauthenticated.') {
          this.$cookie.set('follow_user', this.data.user_id, { expires: '15m' })

          this.$store.commit('login', {
            is_show: true,
            title: '',
            text: ''
          })
        }
      })
    },
  }
}

</script>
