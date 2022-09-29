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
      <Menu :menus="menus" v-if="user_data" />
      <SocialDeck
        v-if="user_data && $route.name === 'private-profile-social'"
        :user_data="user_data"
        :is_private="true"
      />
      <Posts
        v-if="user_data && ($route.name === 'private-profile-me')"
        :user_data="user_data"
        :is_private="true"
      />
      <StoreFront
        v-else-if="user_data && $route.name === 'private-profile-storefront'"
        :user_data="user_data"
        :is_private="true"
      />
      <Tagged
        v-else-if="user_data && $route.name === 'private-profile-tagged'"
        :user_data="user_data"
        :is_private="true"
      />
      <Liked
        v-else-if="user_data && $route.name === 'private-profile-liked'"
        :user_data="user_data"
        :is_private="true"
      />
      <Links
        v-else-if="user_data && $route.name === 'private-profile-links'"
        :user_data="user_data"
        :is_private="true"
      />
      <SeenIn
        v-else-if="user_data && $route.name === 'private-profile-seen-in'"
        :user_data="user_data"
        :is_private="true"
      />
      <Likeboards
        v-else-if="user_data && ($route.name === 'private-profile-likeboards' || $route.name === 'private-profile-likeboards-like')"
        :user_data="user_data"
        :is_private="true"
      />
    </div>
    <LoginFooter v-show="!$store.state.is_logged" />
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
import Liked from '@/components/profile/Liked.vue'
import Links from '@/components/profile/Links.vue'
import Likeboards from '@/components/profile/Likeboards.vue'
import SeenIn from '@/components/profile/SeenIn.vue'
import Loading from '@/components/Loading.vue'
import LoginFooter from '@/components/profile/LoginFooter.vue'

export default {
  name: 'MyProfile',

  components: {
    Header,
    Menu,
    SocialDeck,
    Posts,
    StoreFront,
    Tagged,
    Liked,
    Links,
    Likeboards,
    SeenIn,
    Loading,
    LoginFooter
  },

  data () {
    return {
      menus: [
        {
          text: 'Posts',
          link: '/me',
          type: 'icon',
          icon: 'social-deck',
        },
        {
          text: 'Shop',
          link: '/me/storefront',
          type: 'icon',
          icon: 'shopbag',
        },
      ],
      user_data: false,
      isLogin: false,
      is_connected: false,
    }
  },


  mounted () {
    if (this.$route.name === 'private-profile-seen-in') {
      this.$router.replace('/notifications/engagement');
    }
        
    if (this.$store.state.is_logged) {
        let api = `/api/user/${this.$store.state.user.id}`
        this.$ajax.get(api).then(res => {
            this.user_data = res.data;
            // console.log(this.user_data.profiles.user.unique_key)

            this.$store.commit('setCustomID', '00' + this.user_data.profiles.user.unique_key)

            let social_deck = Object.assign({}, this.user_data.social_accounts);
            this.is_connected = social_deck.length === 0 || Object.keys(social_deck).length === 0 ? false : true;
        });
    } else {
      this.$router.replace('/home/login')
    }
  },

}

</script>
