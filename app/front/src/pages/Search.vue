<template>
  <div>
    <div class="input-wrapper">
      <router-link :to="getPreviousPage()">
        <arrow-left-icon class="icon"/>
      </router-link>

      <input
        v-model="searchedAdress"
        @input="getList"
        :placeholder="$t('search.search')"
        type="text"
        class="search-input"
        v-focus
      >

      <search-icon class="icon"/>

      <div slot="suggestion-item" slot-scope="{ suggestion, query }" class="adress-suggestion">
        <div>{{ suggestion.place_name }}</div>
      </div>
    </div>
    <ul class="results">
      <li
        v-for="address in addresses"
        :key="address.id"
        @click="selectAddress(address)"
      >{{ address.place_name }}</li>
      <search-icon v-if="!addresses.length" class="background"/>
    </ul>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import { ArrowLeftIcon, SearchIcon } from 'vue-feather-icons'

export default {
  name: 'Search',
  components: {
    ArrowLeftIcon,
    SearchIcon
  },
  data() {
    return {
      searchedAdress: null,
      mapboxKey: process.env.MAPBOX_KEY,
      addresses: []
    }
  },
  methods: {
    ...mapActions(['setAddress']),
    getList() {
      if (this.searchedAdress.length > 3) {
        return this.axios
          .get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${this.searchedAdress}.json?access_token=${
              this.mapboxKey
            }`
          )
          .then(resp => {
            this.addresses = resp.data.features
          })
      }
    },
    selectAddress(address) {
      this.setAddress(address)
      this.$router.push({ name: 'Home', query: { l: this.$store.state.selectedAddress.position.join(',') } })
    },
    getPreviousPage() {
      const previousHome = this.$store.state.navigation.routes[this.$store.state.navigation.routes.length - 2]
      return previousHome ? `/?VNK=${previousHome.replace('Home?', '')}` : '/'
    }
  }
}
</script>


<style lang="scss" scoped>
@import '../app.scss';

a {
  color: #fff;

  &:hover {
    color: #fff;
  }
}

.input-wrapper {
  background-color: $mainColor;
  color: #fff;
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 70px;

  .search-input {
    height: 40px;
    box-shadow: none;
    border-radius: 5px;
  }

  .icon {
    height: $iconSize;
    width: $iconSize;
  }
}

.adress-suggestion {
  padding: 10px;
}

.results {
  list-style: none;
  padding: 0;

  li {
    cursor: pointer;
    padding: 20px 20px;
    transition: 0.3s;

    &:hover {
      background-color: #ededed;
    }
  }

  li + li {
    border-top: 1px solid #ccc;
  }

  .background {
    position: absolute;
    top: 20%;
    left: 50%;
    height: 50%;
    width: 50%;
    margin-left: -25%;
    color: #ededed;
  }
}
</style>
