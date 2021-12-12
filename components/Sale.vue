<template>
  <div class="max-w-xl w-full mx-auto px-6 lg:px-8 relative">
    <transition name="fade">
      <div v-if="modal.showing" :class="classModal" style="top: -80px">
        <span class="font-bold mr-2">{{ modal.title }}</span>
        {{ modal.content }}
      </div>
    </transition>
    <div
      class="bg-white border-4 border-box dark:bg-gray-900 overflow-hidden shadow dark:shadow-none rounded-xl p-6"
    >
      <h2 class="font-black dark:text-gray-300 text-gray-900 flex items-center">
        Cocoa DAO Pre-Sale <img src="/building.png" class="ml-2 h-4" />
      </h2>

      <div class="mt-4 flex flex-col">
        <div class="flex justify-between items-end">
          <span class="text-gray-800 dark:text-gray-400">Amount:</span>
          <span class="text-gray-800 dark:text-gray-400 text-xs"
            >Max:
            {{
              whitelist.includes(account)
                ? allocated.whitelist
                : allocated.public
            }}
            $MIM
          </span>
        </div>
        <div class="flex mt-2">
          <div class="flex-grow relative">
            <input
              v-model="invested"
              :disabled="
                !isConnected ||
                (isConnected && mimAllowance === 0) ||
                dates.start > Date.now() ||
                !(dates.whitelist <= Date.now() || whitelist.includes(account))
              "
              type="text"
              placeholder="1,000"
              class="bg-blue-50 disabled:opacity-30 dark:bg-gray-800 rounded-lg text-gray-800 px-4 py-3 h-full w-full focus:outline-none focus:ring"
              @keypress.enter="pay()"
              @keypress="checkTyping"
            />
            <div
              class="inset-y-0 right-0 absolute mx-4 flex justify-center items-center"
            >
              <button
                :disabled="
                  !isConnected ||
                  (isConnected && mimAllowance === 0) ||
                  dates.start > Date.now() ||
                  !(
                    dates.whitelist <= Date.now() || whitelist.includes(account)
                  )
                "
                class="text-blue-600 hover:text-blue-700 focus:text-blue-700 text-sm disabled:opacity-30"
                @click="setMax()"
              >
                MAX
              </button>
            </div>
          </div>
          <div
            class="flex justify-center items-center text-lg py-3 px-2 h-full ml-2 dark:border-gray-700 border rounded-lg cursor-pointer"
          >
            <img src="/mim.png" class="h-6 mr-2" alt="" />
            <span class="font-semibold text-gray-600 dark:text-gray-500"
              >MIM</span
            >
          </div>
        </div>
      </div>
      <div v-if="invested && invested != 0" class="mt-4 flex flex-col text-xs">
        <div class="flex justify-between my-1">
          <span class="font-semibold text-gray-700"> Pay </span>
          <span class="font-bold text-gray-900 flex items-center">
            {{ investedNumber }} <img src="/mim.png" class="h-4 ml-1"
          /></span>
        </div>
        <div class="flex justify-between my-1">
          <span class="font-semibold text-gray-700"> Receive </span>
          <div class="font-bold text-gray-900">
            {{ parseFloat(invested) / 50 }}
            <span class="font-bold text-gray-900"> $cCOCA</span>
          </div>
        </div>
      </div>
      <template v-if="isConnected">
        <template v-if="dates.start <= Date.now()">
          <template
            v-if="dates.whitelist <= Date.now() || whitelist.includes(account)"
          >
            <button
              v-if="mimAllowance === 0"
              class="w-full mt-4 bg-button hover:bg-button-interact focus:bg-button-interact focus:outline-none text-white p-4 rounded-lg"
              @click="addAllowance()"
            >
              Approve Transaction
            </button>
            <button
              v-if="mimAllowance !== 0"
              class="w-full mt-4 bg-button hover:bg-button-interact focus:bg-button-interact focus:outline-none text-white p-4 rounded-lg"
              @click="pay()"
            >
              Buy tokens
            </button>
          </template>
          <template v-else>
            <button
              v-tooltip="'You are not whitelisted'"
              class="w-full mt-4 bg-red-600 text-white p-4 font-bold rounded-lg cursor-default"
            >
              {{ timeRemainingWhitelistString }}
            </button>
          </template>
        </template>
        <template v-else>
          <button
            v-tooltip="
              whitelist.includes(account)
                ? 'The Pre-Sale didn\'t start'
                : 'You are not whitelisted'
            "
            class="w-full mt-4 bg-red-600 text-white p-4 font-bold rounded-lg cursor-default"
          >
            {{
              whitelist.includes(account)
                ? timeRemainingPresaleString
                : timeRemainingWhitelistString
            }}
          </button>
        </template>
      </template>
      <template v-else>
        <button
          class="w-full mt-4 bg-button hover:bg-button-interact focus:bg-button-interact focus:outline-none text-white p-4 rounded-lg"
          @click="connectWallet()"
        >
          Connect Wallet
        </button>
      </template>
    </div>
    <div
      class="mt-6 flex justify-between font-bold text-white dark:text-gray-400"
    >
      <span>{{ selledToken }} $</span> <span>{{ selledCap }} $</span>
    </div>
    <div
      class="overflow-hidden h-3 mb-4 text-xs flex rounded-lg bg-white dark:bg-gray-700 mt-2"
    >
      <div
        :style="styleSale"
        class="shadow-none flex flex-col text-center whitespace-nowrap text-white justify-center bg-green-500"
      ></div>
    </div>
  </div>
</template>

<script>
const { ethers } = require('ethers')

// PARAMS
const getMimContract = (provider) => {
  return new ethers.Contract(
    '0x130966628846BFd36ff31a822705796e8cb8C18D',
    [
      'function balanceOf(address) view returns (uint)',
      'function allowance(address, address) public view returns (uint256)',
      'function approve(address, uint256) public returns (bool)',
      'event Approval(address indexed owner, address indexed spender, uint256 value)',
    ],
    provider
  ).connect(provider.getSigner())
}

// PARAMS
const presaleAddress = '0xd284F5Ae0471A0e2EB8047673Abb35Ae5FD671a4'

// PARAMS
const getPreSaleContract = (provider) => {
  return new ethers.Contract(
    presaleAddress,
    [
      'function presale(uint256 mimAmount) external',
      'function balanceOf(address) view returns (uint)',
      'function totalRaised() public view returns(uint amount)',
    ],
    provider
  ).connect(provider.getSigner())
}

const getBalance = (value, fixedTo = 6) => {
  const puissance = 18 - fixedTo < 0 ? 18 : 18 - fixedTo
  let price = value
    .div(ethers.BigNumber.from(10).pow(ethers.BigNumber.from(puissance)))
    .toString()
  if (price.length < fixedTo || price.length === fixedTo) {
    const diff = fixedTo - price.length
    for (let i = 0; i < diff; i++) {
      price = `0${price}`
    }
    return `0.${price}`
  } else {
    const diff = price.length - fixedTo
    return `${price.substring(0, diff)}.${price.substring(diff)}`
  }
}

const getTime = (time) => {
  let reamaining = time
  const days = parseInt(`${reamaining / (3600 * 1000 * 24)}`)
  reamaining = reamaining - days * (3600 * 1000 * 24)

  let hours = parseInt(`${reamaining / (3600 * 1000)}`)
  reamaining = reamaining - hours * (3600 * 1000)
  if (hours <= 9) {
    hours = `0${hours}`
  }

  let minutes = parseInt(`${reamaining / (60 * 1000)}`)
  reamaining = reamaining - minutes * (60 * 1000)
  if (minutes <= 9) {
    minutes = `0${minutes}`
  }

  let seconds = parseInt(`${reamaining / 1000}`)
  if (seconds <= 9) {
    seconds = `0${seconds}`
  }

  if (days === 0) {
    return `${hours}:${minutes}:${seconds}`
  } else {
    return `${days}d ${hours}:${minutes}:${seconds}`
  }
}

export default {
  props: {
    address: {
      type: Function,
      required: true,
    },
    connect: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    // PARAMS
    return {
      selled: 0,
      cap: 350000,
      allocated: {
        whitelist: 2000,
        public: 1000,
      },
      isConnected: false,
      account: '',
      mimBalance: 0,
      mimAllowance: 0,
      invested: undefined,
      payed: 0,
      whitelist: [],
      dates: {
        start: 1639332000000,
        whitelist: 1639332000000 + 3600 * 24 * 1000,
      },
      timeRemainingPresaleString: '',
      timeRemainingWhitelistString: '',
      modal: {
        showing: false,
        success: false,
        title: 'Error',
        content: 'Default error !',
      },
    }
  },
  async fetch() {
    const whitelistText = await this.$axios.$get('https://cocoadao.finance/whitelist.txt')
    const temp = whitelistText.split(`\n`)
    this.whitelist = temp.map((address) => {
      return address.toLowerCase()
    })
  },
  computed: {
    selledToken() {
      return `${this.selled.toFixed(2)}`.replace(
        /(\d)(?=(\d\d\d)+(?!\d))/g,
        '$1,'
      )
    },
    selledCap() {
      return `${this.cap.toFixed(0)}`.replace(/(\d)(?=(\d\d\d)+(?!\d))/g, '$1,')
    },
    investedNumber() {
      return `${parseFloat(this.invested).toFixed(2)}`.replace(
        /(\d)(?=(\d\d\d)+(?!\d))/g,
        '$1,'
      )
    },
    styleSale() {
      return `width: ${(this.selled / this.cap) * 100}%`
    },
    classModal() {
      if (this.modal.success) {
        return `px-4 py-2 bg-green-400 text-white rounded absolute inset-x-0 mx-6`
      } else {
        return `px-4 py-2 bg-red-400 text-white rounded absolute inset-x-0 mx-6`
      }
    },
  },
  watch: {
    connect(newVal, _) {
      if (newVal === true) {
        this.connectWallet()
      }
    },
  },
  mounted() {
    if (typeof window.ethereum !== 'undefined') {
      this.connectWallet()
    }

    const timeRemainingPresale = this.dates.start - Date.now()
    if (timeRemainingPresale > 0) {
      setTimeout(() => {
        this.dates.start = this.dates.start - 1
      }, timeRemainingPresale + 1)
    }

    const timeRemainingWhitelist = this.dates.whitelist - Date.now()
    if (timeRemainingWhitelist > 0) {
      setTimeout(() => {
        this.dates.whitelist = this.dates.whitelist - 1
      }, timeRemainingWhitelist + 1)
    }

    ;(() => {
      const timeRemainingWhitelist = this.dates.whitelist - Date.now()
      if (timeRemainingWhitelist > 0) {
        this.timeRemainingWhitelistString = getTime(timeRemainingWhitelist)
      }
    })()

    setInterval(() => {
      const timeRemainingWhitelist = this.dates.whitelist - Date.now()
      if (timeRemainingWhitelist > 0) {
        this.timeRemainingWhitelistString = getTime(timeRemainingWhitelist)
      }
    }, 1000)
    ;(() => {
      const timeRemainingPreSale = this.dates.start - Date.now()
      if (timeRemainingPreSale > 0) {
        this.timeRemainingPresaleString = getTime(timeRemainingPreSale)
      }
    })()

    setInterval(() => {
      const timeRemainingPreSale = this.dates.start - Date.now()
      if (timeRemainingPreSale > 0) {
        this.timeRemainingPresaleString = getTime(timeRemainingPreSale)
      }
    }, 1000)
  },
  methods: {
    async syncSelled() {
      const provider = new ethers.providers.Web3Provider(window.ethereum)
      const presaleContract = getPreSaleContract(provider)

      const raised = await presaleContract.totalRaised()
      this.selled = parseFloat(getBalance(raised))
    },
    initializeLoop() {
      this.syncSelled()
      setInterval(() => {
        this.syncSelled()
      }, 60000)
    },
    async pay() {
      if (
        this.invested &&
        this.invested !== 0 &&
        this.whitelist.includes(this.account)
      ) {
        if (this.whitelist.includes(this.account)) {
          if (this.invested > this.allocated.whitelist) {
            this.setModal(
              false,
              'Transaction',
              `You can't exceed more than ${this.allocated.whitelist} $MIM`
            )
            return
          }
        } else if (this.invested > this.allocated.public) {
          this.setModal(
            false,
            'Transaction',
            `You can't exceed more than ${this.allocated.public} $MIM`
          )
          return
        }
        const provider = new ethers.providers.Web3Provider(window.ethereum)
        const presaleContract = getPreSaleContract(provider)
        try {
          await presaleContract.presale(
            ethers.BigNumber.from(this.invested * 10 ** 6).mul(
              ethers.BigNumber.from(10).pow(ethers.BigNumber.from(12))
            )
          )
          this.invested = undefined
          const intervalId = setInterval(async () => {
            const preSaleBalance = await presaleContract.balanceOf(this.account)
            const before = this.payed
            this.payed = parseFloat(getBalance(preSaleBalance))

            if (this.payed > before) {
              this.setModal(true, 'Payment', 'You received your cCOCA')
              this.syncSelled()
              clearInterval(intervalId)
            }
          }, 4000)
          this.setModal(true, 'Transaction', 'Transaction has been sent !')
        } catch (error) {
          if (error.error?.code === -32603) {
            if (
              error.error?.message === 'execution reverted: MIM Amount exceed'
            ) {
              this.setModal(
                false,
                'Transaction',
                "You can't exceed more than 2000 $MIM"
              )
            } else {
              this.setModal(false, 'Transaction', error.error?.message)
            }
          } else if (error?.code === 4001) {
            if (
              error?.message ===
              'MetaMask Tx Signature: User denied transaction signature.'
            ) {
              this.setModal(
                false,
                'Transaction',
                'You must approve the transaction !'
              )
            } else {
              this.setModal(false, 'Transaction', error?.message)
            }
          } else {
            this.setModal(false, 'Transaction', 'Transaction failed !')
          }
        }
      } else {
        this.setModal(false, 'Error', 'Please enter a valid value !')
      }
    },
    async addAllowance() {
      const provider = new ethers.providers.Web3Provider(window.ethereum)
      const mimContract = getMimContract(provider)
      try {
        await mimContract.approve(
          presaleAddress,
          ethers.BigNumber.from('10000').mul(
            ethers.BigNumber.from(10).pow(ethers.BigNumber.from(18))
          )
        )
        const intervalId = setInterval(async () => {
          const mimAllowance = await mimContract.allowance(
            this.account,
            presaleAddress
          )
          this.mimAllowance = parseFloat(getBalance(mimAllowance))

          if (this.mimAllowance > 0) {
            this.setModal(true, 'Transaction', 'You just approved transfer')
            clearInterval(intervalId)
          }
        }, 4000)

        this.setModal(true, 'Transaction', 'Transaction has been sent !')
      } catch (error) {
        this.setModal(false, 'Transaction', 'Transaction failed !')
      }
    },
    async connectWallet() {
      try {
        const accounts = await ethereum.request({
          method: 'eth_requestAccounts',
        })
        const chainId = await ethereum.request({ method: 'eth_chainId' })
        const goodChainid = '0xa86a'
        if (chainId === goodChainid) {
          this.account = accounts[0]
          this.address(this.account)
          ethereum.on('accountsChanged', (accounts) => {
            if (accounts[0] === undefined) {
              window.location.reload()
            } else {
              this.account = accounts[0]
              this.address(this.account)
            }
          })
          ethereum.on('chainChanged', () => {
            window.location.reload()
          })
          this.isConnected = true
          this.setModal(true, 'Connected', "You're connected !")

          const provider = new ethers.providers.Web3Provider(window.ethereum)
          const mimContract = getMimContract(provider)
          const presaleContract = getPreSaleContract(provider)

          const mimBalance = await mimContract.balanceOf(this.account)
          const mimAllowance = await mimContract.allowance(
            this.account,
            presaleAddress
          )

          const preBalance = await presaleContract.balanceOf(this.account)

          this.mimBalance = parseFloat(getBalance(mimBalance))
          this.mimAllowance = parseFloat(getBalance(mimAllowance))
          this.payed = parseFloat(getBalance(preBalance))
          this.initializeLoop()
        } else {
          try {
            await window.ethereum.request({
              method: 'wallet_switchEthereumChain',
              params: [{ chainId: goodChainid }],
            })
          } catch (switchError) {
            if (switchError.code === 4902) {
              try {
                await ethereum.request({
                  method: 'wallet_addEthereumChain',
                  params: [
                    {
                      // PARAMS
                      chainId: goodChainid,
                      chainName: 'Avalanche',
                      nativeCurrency: { name: 'AVAX', symbol: 'AVAX', decimals: 18 },
                      rpcUrls: ['https://api.avax.network/ext/bc/C/rpc'],
                      blockExplorerUrls: ['https://snowtrace.io/'],
                    },
                  ],
                })
              } catch (addError) {
                this.setModal(false, 'Chain Id', 'Wrong Network')
              }
            } else {
              this.setModal(false, 'Chain Id', 'Wrong Network')
            }
          }
        }
      } catch (error) {
        if (error.code === 4001) {
          this.setModal(
            false,
            'MetaMask',
            'To gain access to the presale, you must first connect your wallet',
            5000
          )
        } else {
          this.setModal(
            false,
            'MetaMask',
            'Please download and install MetaMask!',
            5000
          )
        }
      }
    },
    async setMax() {
      if (this.isConnected) {
        const provider = new ethers.providers.Web3Provider(window.ethereum)
        const mimContract = getMimContract(provider)

        const mimBalance = await mimContract.balanceOf(this.account)
        const mimAllowance = await mimContract.allowance(
          this.account,
          presaleAddress
        )

        this.mimBalance = parseFloat(getBalance(mimBalance))
        this.mimAllowance = parseFloat(getBalance(mimAllowance))

        let balance = 0

        if (this.mimBalance < this.mimAllowance) {
          balance = this.mimBalance
        } else {
          balance = this.mimAllowance
        }

        if (this.whitelist.includes(this.account)) {
          if (balance > this.allocated.whitelist) {
            this.invested = this.allocated.whitelist
          } else {
            this.invested = balance
          }
        } else if (balance > this.allocated.public) {
          this.invested = this.allocated.public
        } else {
          this.invested = balance
        }
      }
    },

    checkTyping(e) {
      const allowed = [
        '0',
        '1',
        '2',
        '3',
        '4',
        '5',
        '6',
        '7',
        '8',
        '9',
        '.',
        ',',
      ]

      if (!allowed.includes(e.key)) {
        e.preventDefault()
      }
    },

    setModal(success, title, content, time = 3000) {
      this.modal.success = success
      this.modal.content = content
      this.modal.title = title
      this.modal.showing = true
      setTimeout(() => {
        this.modal.showing = false
      }, time)
    },
  },
}
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
