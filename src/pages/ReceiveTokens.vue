<template>
  <q-page>
    <section class="header-bg" />
    <section
      class="body-container"
      style="max-width: 580px"
      v-if="!isAuthenticated"
    >
      <q-card class="not-authenticated">
        <div class="text-subtitle1">Please login</div>
      </q-card>
    </section>
    <section class="body-container" style="max-width: 580px" v-else>
      <q-card class="authenticated">
        <!-- Back button -->
        <q-btn
          :to="{ name: 'wallet', params: { accountName: accountName } }"
          flat
          round
          class="self-start"
        >
          <q-icon name="fas fa-chevron-circle-left" style="font-size: 50px" />
        </q-btn>
        <!-- Select token dropdown -->
        <div class="column items-center">
          <q-btn-dropdown
            no-caps
            flat
            class="q-ml-md bg-secondary"
            padding="xs"
            style="margin: 8px"
          >
            <template v-slot:label>
              <div class="flex items-center justify-center wrap q-pa-sm">
                <h2>
                  Receive
                </h2>
                <div class="row items-center justify-center">
                  <token-avatar :token="selectedToken" :avatarSize="55" />
                  <h2>
                    {{ selectedToken }}
                  </h2>
                </div>
              </div>
            </template>
            <q-list class="bg-secondary">
              <q-item
                clickable
                v-close-popup
                v-for="token in tokens"
                :key="token"
                @click="
                  selectedToken = token;
                  selectedNetwork = currentChain.NETWORK_NAME.toLowerCase();
                "
                flat
                size="lg"
                no-caps
                padding="xs"
              >
                <q-item-section avatar>
                  <token-avatar :token="token" :avatarSize="40" />
                </q-item-section>
                <q-item-section>
                  <q-item-label style="font-weight: 500">
                    {{ token }}
                  </q-item-label>
                </q-item-section>
                <q-item-section side>
                  <q-btn
                    icon="fas fa-external-link-alt"
                    class="hover-accent"
                    type="a"
                    target="_blank"
                    :href="pTokenBridgeLink(token)"
                    size="12px"
                    round
                    flat
                  />
                  <q-tooltip>pTokens dapp</q-tooltip>
                </q-item-section>
              </q-item>
            </q-list>
          </q-btn-dropdown>
          <!-- Network selection -->
          <div class="text-subtitle1 q-pb-sm">From network</div>
          <div class="networks row justify-center">
            <q-btn
              :label="currentChain.NETWORK_DISPLAY_NAME"
              @click="selectedNetwork = currentChain.NETWORK_NAME.toLowerCase()"
              :class="
                selectedNetwork === currentChain.NETWORK_NAME.toLowerCase()
                  ? 'selected-network'
                  : ''
              "
              flat
              size="lg"
              no-caps
              padding="xs"
            />
            <q-btn
              label="Bitcoin"
              v-if="selectedToken.toUpperCase() === 'PBTC'"
              @click="selectedNetwork = 'bitcoin'"
              :class="selectedNetwork === 'bitcoin' ? 'selected-network' : ''"
              flat
              size="lg"
              no-caps
              padding="xs"
            />
            <!-- type="a"
              target="_blank"
              :href="pTokenBridgeLink('PBTC')" -->
            <q-btn
              label="Ethereum"
              v-if="['PETH', 'TLOS', 'PUSDC', 'PUSDT', 'EOS'].includes(selectedToken.toUpperCase())"
              @click="selectedNetwork = 'ethereum'"
              :class="selectedNetwork === 'ethereum' ? 'selected-network' : ''"
              flat
              size="lg"
              no-caps
              padding="xs"
            >
              <!-- 
              type="a"
              target="_blank"
                :href="
                selectedToken.toUpperCase() === 'PETH'
                  ? pTokenBridgeLink('PETH')
                  : pTokenBridgeLink('TLOS')
              " -->
              <q-tooltip>pTokens dapp</q-tooltip>
            </q-btn>
          </div>

          <!-- """""""""""""""""""""""""""""" -->
          <!-- Current chain network selected -->
          <!-- """""""""""""""""""""""""""""" -->
          <div
            class=" column items-center"
            v-show="selectedNetwork === currentChain.NETWORK_NAME.toLowerCase()"
          >
            <div class="text-subtitle1 q-py-md text-center">
              Deposit {{ depositTokenStr }} to the following address
            </div>
            <!-- telos qr code-->
            <div></div>
            <div id="tlos-qr-canvas" />

            <!-- <div id="btc-qr-canvas" v-show="selectedNetwork === 'bitcoin'" /> -->
            <!-- Address info -->
            <div class="col text-subtitle1 row q-gutter-x-sm q-mx-md">
              <div
                id="address"
                :class="`col ${selectedNetwork}-net`"
                style="word-wrap: break-word;"
              >
                {{ selectedAddress }}
              </div>
              <!-- Copy address button -->
              <div class="row content-center q-pl-sm">
                <q-btn
                  flat
                  v-if="selectedAddress !== ''"
                  @click="copyAddress(selectedAddress)"
                  icon="far fa-clone"
                  padding="0"
                  color="positive"
                  size="sm"
                />
              </div>
            </div>
          </div>

          <!-- Generate address button -->
          <!-- <q-btn
            class="q-mt-md"
            v-if="selectedNetwork === 'bitcoin'"
            @click="setAddresses"
            color="primary"
            label="Generate new address"
          /> -->
          <!-- Ethereum receive  -->
          <!-- <div
            v-if="selectedNetwork === 'ethereum'"
            class="column items-center q-pt-md q-gutter-y-sm"
          > -->
          <!-- If metamask isn't installed start onboarding process -->
          <!-- <q-btn
              v-if="isMetaMaskInstalled && !metamaskConnected"
              color="primary"
              label="Connect Metamask"
              @click="ethereumConnect()"
            /> -->
          <!-- Else login with metamask -->
          <!-- <div v-if="!isMetaMaskInstalled">
              Install or enable metamask first.
            </div>
            <q-btn
              v-if="!isMetaMaskInstalled"
              color="primary"
              label="Install metamask"
              @click="metamaskOnboarding()"
            /> -->
          <!-- Input amount of eth to peth -->
          <!-- <div
              v-if="
                selectedToken.toUpperCase() === 'PETH' &&
                  isMetaMaskInstalled &&
                  metamaskConnected
              "
              class="column items-center q-gutter-y-md"
            >
              <q-input
                v-model="amount"
                label="ETH"
                outlined
                placeholder="0"
                autofocus
                @keyup.enter="tryPegIn()"
                :disable="devMode"
                :loading="txnPending"
              />
              <q-btn
                color="primary"
                label="Issue"
                @click="tryPegIn()"
                :disable="devMode || txnPending === true"
              />
            </div> -->
          <!-- Input amount of tlos erc20 to tlos -->
          <!-- <div
              v-if="
                selectedToken.toUpperCase() === 'TLOS' &&
                  isMetaMaskInstalled &&
                  metamaskConnected
              "
              class="column items-center q-gutter-y-md"
            >
              <q-input
                v-model="amount"
                label="TLOS (ERC-20)"
                outlined
                placeholder="0"
                autofocus
                @keyup.enter="tryPegOut()"
                :disable="devMode"
                :loading="txnPending"
              />
              <q-btn
                color="primary"
                label="Redeem"
                @click="tryPegOut()"
                :disable="devMode || txnPending === true"
              />
            </div>
          </div> -->
          <!-- TODO make this pretty -->
          <!-- <q-linear-progress
            indeterminate
            size="25px"
            color="primary"
            track-color="secondary"
            rounded
            style="margin-top: 15px"
            v-if="txnPending"
          >
            <div class="absolute-full flex flex-center">
              <q-badge
                text-color="black"
                color="secondary"
                :label="txStatusMessage"
              />
            </div>
          </q-linear-progress> -->
        </div>
      </q-card>

      <!-- pNetwork app -->
      <!-- TODO This causes a DOM error, but hey it works. -->
      <div
        class="q-mt-md q-card bg-white"
        style="padding: 0 0 20px 0"
        v-if="selectedNetwork.toUpperCase() != currentChain.NETWORK_NAME"
      >
        <div style="border-radius: 10px; overflow: hidden;">
          <iframe
            height="720"
            width="100%"
            :src="pTokenBridgeLink(selectedToken)"
            allowfullscreen
            frameBorder="0"
          >
          </iframe>
        </div>
      </div>

      <!-- Testnet warning -->
      <!-- <q-banner
        rounded
        inline-actions
        class="text-white bg-negative"
        style="margin-top: 15px"
        v-if="onTestnet"
      >
        Do not make real payments! This is a testnet.
      </q-banner> -->
    </section>
  </q-page>
</template>

<script>
import axios from "axios";
import { mapGetters, mapActions } from "vuex";
import QRCodeStyling from "qr-code-styling";
import QRCode from "qrcode";
import { copyToClipboard } from "quasar";
import tokenAvatar from "src/components/TokenAvatar";
import { PBTC } from "ptokens-pbtc";
import { pERC20 } from "ptokens-perc20";
import { pEosioToken } from "ptokens-peosio-token";
import { HttpProvider } from "ptokens-providers";
import { Node } from "ptokens-node";
import Web3 from "web3";
import MetaMaskOnboarding from "@metamask/onboarding";
import { BigNumber } from "bignumber.js";
import { constants, eth } from "ptokens-utils";

const qrStyling = {
  data: "",
  qrOptions: {
    typeNumber: "0",
    mode: "Byte",
    errorCorrectionLevel: "Q"
  },
  dotsOptions: {
    type: "rounded",
    color: "black"
  },
  cornersSquareOptions: { type: "extra-rounded", color: "black" },
  cornersSquareOptionsHelper: {
    colorType: { single: true, gradient: false },
    gradient: {
      linear: true,
      radial: false,
      color1: "black",
      color2: "black",
      rotation: "0"
    }
  },
  cornersDotOptions: { type: "", color: "black" },
  cornersDotOptionsHelper: {
    colorType: { single: true, gradient: false },
    gradient: {
      linear: true,
      radial: false,
      color1: "black",
      color2: "black",
      rotation: "0"
    }
  },
  backgroundOptions: {
    color: "#f9fbfe"
  },
  imageOptions: {
    crossOrigin: "anonymous",
    margin: 20
  }
};

export default {
  components: { tokenAvatar },
  data() {
    return {
      // devMode: Boolean(process.env.DEVELOPMENT),
      devMode: false,
      receiveLink: "",
      qrCodes: {
        tlos: new QRCodeStyling({ width: 180, height: 180, ...qrStyling }),
        btc: new QRCodeStyling({ width: 270, height: 270, ...qrStyling })
      },
      btcAddress: "",
      selectedNetwork: "telos",
      selectedToken: "PBTC",
      ethAccounts: [],
      amount: 0,
      txnPending: false,
      txStatusMessage: ""
    };
  },
  computed: {
    ...mapGetters("account", ["isAuthenticated", "accountName", "wallet"]),
    ...mapGetters("blockchains", ["currentChain"]),
    onTestnet() {
      return process.env.TESTNET;
    },

    tokens() {
      if (this.currentChain.NETWORK_NAME === "TELOS") {
        return ["PBTC", "PETH", "TLOS", "PUSDC", 'PUSDT'];
      } else if (this.currentChain.NETWORK_NAME === "EOS") {
        return ["PBTC", "PETH", 'EOS'];
      } else {
        return ["PBTC", "PETH", "TLOS"]
      }
    },

    depositTokenStr() {
      if (this.selectedToken.toUpperCase() === "PETH") {
        if (this.selectedNetwork === "telos") return "PETH";
        else return "ETH";
      } else if (this.selectedToken.toUpperCase() === "TLOS") {
        if (this.selectedNetwork === "telos") return "TLOS";
        else return "TLOS (ERC20)";
      } else {
        if (this.selectedNetwork === "telos") return "PBTC";
        else return "BTC";
      }
    },
    selectedAddress() {
      if (this.selectedNetwork === this.currentChain.NETWORK_NAME.toLowerCase())
        return this.accountName;
      else return this.btcAddress;
    },

    selectedEthAccount() {
      return window.ethereum.selectedAddress;
    },

    //Created check function to see if the MetaMask extension is installed
    isMetaMaskInstalled() {
      //Have to check the ethereum binding on the window object to see if it's installed
      const { ethereum } = window;
      return Boolean(ethereum && ethereum.isMetaMask);
    },

    metamaskConnected() {
      if (window.ethereum.selectedAddress > 0 || this.ethAccounts.length > 0) {
        return true;
      } else {
        return false;
      }
    }
  },

  methods: {
    pTokenBridgeLink(sym) {
      sym = sym.toUpperCase();
      // if telos chain
      if (this.currentChain.NETWORK_NAME === "TELOS") {
        if (sym === "TLOS") {
          return "https://dapp.ptokens.io/swap?asset=tlos&from=eth&to=telos";
        } else if (sym === "PBTC") {
          return "https://dapp.ptokens.io/swap?asset=btc&from=btc&to=telos";
        } else if (sym === "PETH") {
          return "https://dapp.ptokens.io/swap?asset=eth&from=eth&to=telos";
        } else if (sym === "PUSDC") {
          return "https://dapp.ptokens.io/swap?asset=usdc&from=eth&to=telos";
        } else if (sym === "PUSDT") {
          return "https://dapp.ptokens.io/swap?asset=usdt&from=eth&to=telos";
        }
      } else if (this.currentChain.NETWORK_NAME === "EOS") {
        if (sym === "PBTC") {
          return "https://dapp.ptokens.io/swap?asset=btc&from=btc&to=eos";
        } else if (sym === "PETH") {
          return "https://dapp.ptokens.io/swap?asset=eth&from=eth&to=eos";
        } else if (sym === "EOS") {
          return "https://dapp.ptokens.io/swap?asset=eos&from=eth&to=eos";
        }
      } else if (this.currentChain.NETWORK_NAME === "WAX") {
        
      }
    },

    copyAddress(adress) {
      copyToClipboard(adress).then(() => {
        this.$q.notify({
          color: "green-4",
          textColor: "secondary",
          message: "Copied account to clipboard",
          timeout: 1000
        });
      });
    },

    async getCurrentGasPrice() {
      web3 = new Web3(window.ethereum);
      let gas_wei = await web3.eth.getGasPrice();
      return gas_wei;
    },

    toWei(number) {
      return BigNumber(number).multipliedBy(10 ** 18);
    },

    async generateQR(text) {
      try {
        return await QRCode.toDataURL(text);
      } catch (err) {
        console.error(err);
      }
    },

    async setAddresses() {
      // current chain QR code
      this.qrCodes.tlos.update({ data: this.accountName });
      this.qrCodes.tlos.append(document.getElementById("tlos-qr-canvas"));

      // bitcoin qr code
      // const pbridge_api = await axios.get(
      //   `https://pbtcon${this.currentChain.NETWORK_NAME.toLowerCase()}-node-1a.ngrok.io/pbtc-on-${this.currentChain.NETWORK_NAME.toLowerCase()}/get-native-deposit-address/${
      //     this.accountName
      //   }`
      // );
      // this.btcAddress = pbridge_api.data.nativeDepositAddress || [];
      // this.qrCodes.btc.update({ data: this.btcAddress });      
      // this.qrCodes.btc.append(document.getElementById("btc-qr-canvas"));
    },

    metamaskOnboarding() {
      const onboarding = new MetaMaskOnboarding();
      onboarding.startOnboarding();
    },

    async ethereumConnect() {
      if (window.ethereum) {
        web3 = new Web3(window.ethereum);
        const accounts = await ethereum.request({
          method: "eth_requestAccounts"
        });
        this.ethAccounts = await ethereum.request({ method: "eth_accounts" });
        console.log(this.ethAccounts);
      }
    },

    async tryPegIn() {
      try {
        this.txnPending = true;
        await this.pegIn();
        this.$q.notify({
          color: "green-4",
          textColor: "white",
          icon: "cloud_done",
          message: "Completed"
        });
        this.txnPending = false;
      } catch (error) {
        this.$q.notify({
          color: "red-5",
          textColor: "white",
          icon: "warning",
          message: `${error}`
        });
        this.txnPending = false;
      }
    },

    // ETH to PETH
    async pegIn() {
      if (window.ethereum) {
        this.txStatusMessage = "Preparing for transaction";
        const peth = new pERC20({
          pToken: "PETH",
          ethProvider: window.ethereum,

          hostBlockchain: "telos",
          hostNetwork: "mainnet",
          nativeBlockchain: "ethereum",
          nativeNetwork: "mainnet",

          telosRpc: "https://telos.caleos.io" //  TODO process.env.HYPERION_ENDPOINT, use instead
        });

        try {
          const transaction = await peth
            .issue(this.toWei(this.amount), this.accountName, {
              gas: 200000,
              gasPrice: await this.getCurrentGasPrice()
            })
            .once("nativeTxBroadcasted", tx => {
              this.txStatusMessage = "Native transaction broadcasted";
              console.log(tx);
            })
            .once("nativeTxConfirmed", tx => {
              this.txStatusMessage = "Native transaction confirmed";
              console.log(tx.transactionHash);
              this.txnPending = false;
            });
          // .once("nodeReceivedTx", report => {
          //   this.txStatusMessage = "Node received transaction";
          //   console.log(tx);
          // })
          // .once("nodeBroadcastedTx", report => {
          //   this.txStatusMessage = "Node broadcasted transaction";
          //   console.log(tx);
          // })
          // .once("hostTxConfirmed", tx => {
          //   this.txStatusMessage = "Host transaction confirmed";
          //   console.log(tx);
          // })
          // .then(res => console.log(res));
        } catch (e) {
          throw e.cause.message;
        }
      } else {
        console.log("No web3 detected");
      }
    },

    async tryPegOut() {
      try {
        this.txnPending = true;
        await this.pegOut();
        this.$q.notify({
          color: "green-4",
          textColor: "white",
          icon: "cloud_done",
          message: "Submitted"
        });
        this.txnPending = false;
      } catch (error) {
        this.$errorNotification(error);
        this.txnPending = false;
      }
    },

    //TLOS (ERC20) to TLOS
    async pegOut() {
      this.txStatusMessage = "Preparing for transaction";
      const telos = new pEosioToken({
        pToken: "TLOS",

        // if you want to be more detailed
        hostBlockchain: "ethereum",
        hostNetwork: "mainnet", // possible values are testnet_jungle2, testnet_ropsten and mainnet
        nativeBlockchain: "telos",
        nativeNetwork: "mainnet",

        // optionals
        ethProvider: window.ethereum, // or instance of Web3 provider
        telosRpc: "https://telos.caleos.io" //  TODO process.env.HYPERION_ENDPOINT, use instead
      });

      try {
        const transaction = await telos
          .redeem(this.toWei(this.amount), this.accountName, {
            gasPrice: await this.getCurrentGasPrice(),
            gas: 200000
          })
          .once("hostTxConfirmed", tx => {
            this.txStatusMessage = "Host transaction confirmed";
            console.log(tx.transactionHash);
            this.txnPending = false;
          });
        // .once("nodeReceivedTx", report => {
        //   this.txStatusMessage = "Node received transaction";
        //   console.log(report);
        // })
        // .once("nodeBroadcastedTx", report => {
        //   this.txStatusMessage = "Node broadcasted transaction";
        //   console.log(report);
        // })
        // .once("nativeTxConfirmed", tx => {
        //   this.txStatusMessage = "Transaction Completed";
        //   console.log(tx);
        // })
        // .then(res => console.log(res));
      } catch (e) {
        throw e.cause.message;
      }
    }
  },

  mounted() {
    this.selectedNetwork = this.currentChain.NETWORK_NAME.toLowerCase(); // sets network to current chain
    this.setAddresses(); // makes qr codes
    if (this.$route.query.token_sym !== undefined)
      this.selectedToken = this.$route.query.token_sym;
  }
};
</script>

<style lang="scss" scoped>
.q-card {
  &.not-authenticated {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
    min-height: 100px;
  }
  &.authenticated {
    display: grid;
    align-items: stretch;
    grid-template-columns: 50px auto 50px;
    padding-bottom: 40px;
    & div {
      margin: 0;
      @media only screen and (max-width: 585px) {
        grid-column-start: 1;
        grid-column-end: 4;
      }
    }
  }
}
.header-bg {
  height: 160px;
  margin-bottom: -50px;
}
h2 {
  line-height: 45px;
  margin: 0 10px;
  font-size: 34px;
}
.q-item:hover {
  background-color: $dark;
  color: $secondary;
  & .q-btn {
    color: $secondary;
  }
}
@media only screen and (max-width: 425px) {
  #btc-qr-canvas {
    margin: -30px -20px;
    transform: scale(0.8);
  }
  #address.bitcoin-net {
    width: 200px;
  }
}
.q-input {
  max-width: 200px;
}
.resp-iframe {
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  width: 100%;
  height: 100%;
}
// .resp-container {
// }
</style>
