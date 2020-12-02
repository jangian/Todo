<template>
  <div class="sliderMenu grayBgColor">
    <div class="sliderHeader">
      Online users - {{ online_list.length }}
    </div>
    <div class="userInfo" v-for="user in online_list" v-bind:key="user.user.name">
      <div class="userImg">
        <i class="far fa-user" />
      </div>
      <div class="userName">
        {{ user.user.name }}
      </div>
    </div>
  </div>
</template>

<script>
  import gql from 'graphql-tag'
  import { setInterval } from 'timers'
  const SUBSCRIPTION_ONLINE_USERS = gql`
    subscription getOnlineUsers {
      online_users 
    }
  `

  export default {
    data() {
      return {
        online_list: [
          { user: { name: "someUser1" }},
          { user: { name: "someUser2" }}
        ]
      };
    },
    apollo: {
      $subscribe: {
        online_list: {
          query: SUBSCRIPTION_ONLINE_USERS,
          result(data) {
            this.online_list = data.data.online_users
          }
        }
      }
    }
    mounted() {
      const UPDATE_LASTSEEN_MUTATION = gql`
        mutation updateLastSeen($now: timestamptz!) {
          update_users(where: {}, _set: {last_seen: $now}) {
            affected_rows
          }
        }
      `;
      setInterval(function() {
        this.$apollo
          .mutate({
            mutation: UPDATE_LASTSEEN_MUTATIOn,
            variables: {
              now: new Date().toISOString()
            }
          })
          .catch(error => {
            console.error(error);
          })
      }.bind(this),30000);
    }
  }

</script>
