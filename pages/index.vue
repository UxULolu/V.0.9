<template>
  <div>
    <div class="flex flex min-h-screen custom-background relative md:static">
      <div
        v-if="mobile && sidebar"
        class="absolute h-full w-full z-10 bg-black opacity-50"
        @click="sidebar = false"
      ></div>
      <div
        v-if="!mobile || (mobile && sidebar)"
        class="w-52 bg-white h-full md:h-auto absolute md:static md:flex z-20"
      >
        <div class="flex flex-col w-full h-full">
          <div class="flex justify-center mt-6">
            <div class="h-20 w-20 rounded-full">
              <img src="/logo.png" alt="" />
            </div>
          </div>
          <div class="mt-16"></div>
          <NuxtLink
            to="/"
            class="w-full pl-8 py-2 text-button text-sm hover:text-button hover:bg-gray-100 flex items-center font-bold"
          >
            <i class="bx bx-swim mr-2"></i>OG Pre-Sale<span
              class="bg-button h-1.5 w-1.5 ml-12 rounded-full"
            ></span
          ></NuxtLink>
          <a
            class="w-full pl-8 py-1 font-semibold text-gray-700 cursor-pointer text-sm flex items-center"
            @click="addToken()"
          >
            <i class="bx bx-plus mr-2"></i
            ><span class="text-xs">Add Token</span></a
          >

          <span class="pl-8 mt-12 font-bold">Coming Soon</span>
          <a
            disabled
            class="w-full pl-8 py-2 font-semibold text-gray-400 hover:bg-gray-100 select-none text-sm mt-3 flex items-center"
          >
            <i class="bx bxs-bank mr-2"></i>Bond</a
          >
          <a
            disabled
            class="w-full pl-8 py-2 font-semibold text-gray-400 hover:bg-gray-100 select-none text-sm mt-1"
          >
            <i class="bx bx-coin mr-2"></i>COCA-MIM 0,0%</a
          >
          <a
            disabled
            class="w-full pl-8 py-2 font-semibold text-gray-400 hover:bg-gray-100 select-none text-sm mt-1"
          >
            <i class="bx bx-coin mr-2"></i>MIM 0,0%</a
          >
          <a
            disabled
            class="w-full pl-8 py-2 font-semibold text-gray-400 hover:bg-gray-100 select-none text-sm mt-1"
          >
            <i class="bx bx-lock-alt mr-2"></i>Stake</a
          >
          <footer
            class="py-4 bg-white w-full flex-grow flex flex-col justify-end"
          >
            <ul class="flex justify-between text-3xl px-6 mt-6">
              <a
                href="https://discord.gg/cocoadao"
                class="flex items-center text-black hover:text-button"
              >
                <i class="bx bxl-discord-alt mr-2"></i>
              </a>
              <a
                href="https://twitter.com/Daococoa"
                class="flex items-center text-black hover:text-button"
              >
                <i class="bx bxl-twitter mr-2"></i>
              </a>
              <a
                href="https://medium.com/cocoadao"
                class="flex items-center text-black hover:text-button"
              >
                <i class="bx bxl-medium mr-2"></i>
              </a>
            </ul>
          </footer>
        </div>
      </div>
      <div class="flex flex-col items-top justify-between flex-grow">
        <div class="flex px-6 mt-6 justify-between">
          <button
            v-if="mobile"
            class="h-10 w-10 bg-white rounded"
            @click="sidebar = !sidebar"
          >
            <i class="bx bx-menu"></i>
          </button>
          <div v-else class="h-0 w-0"></div>
          <button
            v-if="address == ''"
            class="px-4 py-2 bg-white rounded-full bg-button text-white"
            @click="connect = true"
          >
            Connect Wallet</button
          ><button
            v-else
            class="px-4 py-2 bg-white rounded-full bg-button text-white"
          >
            {{ showAddress }}
          </button>
        </div>
        <div class="flex-grow flex items-center justify-center">
          <Sale :address="setAddress" :connect="connect" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'

const tokenAddress = '0xd00981105e61274c8a5cd5a88fe7e037d935b513'
const tokenSymbol = 'cCOCA'
const tokenDecimals = 18
const tokenImage = 'http://placekitten.com/200/300'

export default Vue.extend({
  data: () => {
    return {
      sidebar: false,
      mobile: false,
      address: '',
      connect: false,
    }
  },
  computed: {
    showAddress() {
      return `${this.address.substring(0, 6)}...${this.address.substring(
        this.address.length - 4,
        this.address.length
      )}`
    },
  },
  mounted() {
    if (window.innerWidth <= 768) {
      this.mobile = true
    }
    this.$nextTick(() => {
      window.addEventListener('resize', () => {
        if (window.innerWidth <= 768) {
          this.mobile = true
        } else {
          this.mobile = false
        }
      })
    })
  },
  methods: {
    setAddress(account) {
      this.address = account
    },
    async addToken() {
      try {
        await ethereum.request({
          method: 'wallet_watchAsset',
          params: {
            type: 'ERC20', // Initially only supports ERC20, but eventually more!
            options: {
              address: tokenAddress, // The address that the token is at.
              symbol: tokenSymbol, // A ticker symbol or shorthand, up to 5 chars.
              decimals: tokenDecimals, // The number of decimals in the token
              image: tokenImage, // A string url of the token logo
            },
          },
        })
      } catch (error) {}
    },
  },
})
</script>

<style>
html,
body {
  background-color: black;
}

.custom-background {
  background-image: linear-gradient(rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0.3)),
    url('/junglehd.jpeg');
  background-position-y: 25%;
  background-repeat: no-repeat;
  background-size: cover;
}
</style>
