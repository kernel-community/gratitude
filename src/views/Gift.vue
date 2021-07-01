<template>
  <div>
    <div
      class="bg-gradient-to-r from-purple-400 to-blue-500 min-h-screen min-w-screen py-6 flex flex-col justify-center sm:py-12"
    >
      <div class="relative py-3 sm:max-w-xl sm:mx-auto">
        <div
          class="sm:absolute sm:inset-0 sm:bg-gradient-to-r sm:from-purple-500 sm:to-purple-900 sm:shadow-lg sm:transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"
        ></div>
        <div
          class="relative px-1 py-10 bg-gray-50 shadow-lg sm:rounded-3xl sm:p-20"
        >
          <div class="max-w-md mx-auto">
            <div v-if="!fielding.loading">
              <div v-if="fielding.ok">
                <Header
                  :name="fromApi.name"
                  :award="fromApi.award"
                  :tweetText="tweetText"
                />
                <Canvas
                  :fellowName="fromApi.name"
                  class="h-56 w-56 sm:h-96 sm:w-96 shadow-2xl sm:m-5 mx-auto my-5"
                />
                <div
                  v-if="!loading && !fielding.msg.txHash && !fielding.msg.alert"
                  class="sm:m-5 mx-5 grid justify-items-center"
                >
                  <!-- if claim status = false / not already minted -->
                  <div v-if="!$store.getters.claim_status">
                    <!-- display if metamask is not connected -->
                    <div v-if="!$store.getters.account">
                      <button class="kernel-btn" v-on:click="connectMetamask">
                        Connect Metamask
                      </button>
                    </div>
                    <!-- display if metamask is connected -->
                    <div v-else>
                      <button
                        class="kernel-btn"
                        v-on:click="claim"
                        v-if="$store.getters.network == $store.getters.chainId"
                      >
                        Claim
                      </button>

                      <!-- display if the metamask is not connected to the correct network -->
                      <div v-else>
                        <p
                          class="animate-pulse text-sm bg-gradient-to-r from-blue-50 via-blue-100 to-blue-50 py-5 px-5 shadow-md rounded-xl m-5 ring-4 ring-blue-100 font-lato font-light"
                        >
                          <a
                            class="cursor-pointer"
                            v-on:click="switchNetwork"
                            v-if="network == 'matic' || network == 'mumbai'"
                            >Click here to connect</a
                          >
                          <span v-else>Switch</span>
                          to
                          <span class="font-semibold">{{ network }}</span>
                          network and reload your window.
                        </p>
                      </div>
                    </div>
                  </div>
                  <!-- if already minted -->
                  <div v-else class="items-start">
                    <p class="truncate">
                      This token is minted to:
                      <span class="font-mono truncate">
                        <a :href="linkTo.tokens" target="_new">
                          {{
                            $store.getters.currentOwner.substring(0, 8) + ".."
                          }}
                        </a>
                      </span>
                    </p>
                  </div>
                </div>
                <Spinner v-else-if="loading == true" />

                <!-- minting tx sent info text -->
                <PulsingInfoText
                  v-if="fielding.msg.txHash && !loading"
                  text="Minting transaction sent"
                  :transactionHash="fielding.msg.txHash"
                  :linkToTxHash="linkTo.txHash"
                />
                <!-- alert info text -->
                <PulsingInfoText
                  v-if="fielding.msg.alert && !loading"
                  :text="fielding.msg.alert"
                />

                <!-- stuff in the bottom of the card -->
                <div class="p-5 sm:p-0 grid justify-items-start">
                  <!-- view testimonials button -->
                  <ViewTestimonialsButton />
                  <!-- footer -->
                  <Footer />
                </div>
              </div>

              <!-- incorrect hash supplied -->
              <div v-else>
                <p class="sm:py-3 text-3xl p-3 text-center">
                  {{ fielding.notAuthMsg }}
                </p>
              </div>
            </div>
            <!-- full page loading -->
            <Spinner v-else />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Canvas from "../components/sections/Canvas";
import Header from "../components/sections/Header";
import Spinner from "../components/Spinner.vue";
import Footer from "../components/sections/Footer.vue";
import ViewTestimonialsButton from "../components/ViewTestimonialsButton.vue";
import PulsingInfoText from "../components/PulsingInfoText.vue";

const axios = require("axios");
const Web3 = require("web3");
const contractAssets = require("../assets/contract");
const contractNetwork = contractAssets[contractAssets.active];
const web3 = new Web3(contractNetwork.rpc);
const config = require("../config");
const { env } = config;
const apiUrl = config[env]["server"];
const contract = new web3.eth.Contract(
  JSON.parse(contractAssets.abi),
  contractNetwork.address
);

export default {
  props: {
    hash: String
  },
  components: {
    Canvas,
    Header,
    Spinner,
    Footer,
    ViewTestimonialsButton,
    PulsingInfoText
  },
  data: () => {
    return {
      fromApi: {
        name: "",
        token: "",
        proof: [],
        award: "",
        gratitude: {},
        image: ""
      },
      fielding: {
        loading: false,
        notAuthMsg: "",
        ok: false,
        msg: {}
      },
      linkTo: {
        tokens: "",
        txHash: ""
      },
      loading: false,
      isOpen: true,
      network: contractAssets.active,
      tweetText: ""
    };
  },
  beforeCreate: async function() {
    this.$store.commit("change", {
      chainId: contractNetwork.chainId
    });
  },
  created: async function() {
    this.$data.fielding.loading = true;
    // fetch hash details
    let r = await axios.get(apiUrl + "/gift/hash/" + this.$props.hash);

    // details fetched -> set loading to false
    this.$data.fielding.loading = false;
    const { ok } = r.data;

    // pick and display info
    if (ok) {
      this.$data.fielding.ok = true;
      const { name, token } = r.data.data.details;
      this.$store.commit("addToken", token);
      const { proof } = r.data.data;
      const { image } = r.data.data.metadata;
      const { gratitude, award } = r.data.data.metadata;
      this.$data.fromApi = {
        name,
        token,
        proof,
        award,
        gratitude,
        image
      };
      let tweet =
        "Just received a token of web3 appreciation (as non-fungible as me 😉) for helping co-create @KERNEL0x.\n\nCheck out my kernels 🌱: https://gratitude.kernel.community" +
        this.$route.fullPath;
      this.$data.tweetText =
        "http://twitter.com/intent/tweet?text=" + encodeURIComponent(tweet);
    } else {
      this.$data.fielding.notAuthMsg = "Access is by Invitation only";
    }

    // change connected address, network and expected chain id from metamask and contract json
    this.$store.commit("change", {
      account: this.ethereum.selectedAddress,
      network: this.ethereum.networkVersion,
      chainId: contractNetwork.chainId
    });
    this.$data.linkTo.tokens = contractNetwork.addressLink.replace(
      "<address>",
      this.$store.getters.account
    );
    // check claim status
    let ownerOf = contract.methods.ownerOf(this.$data.fromApi.token.toString());
    let currentOwner;
    try {
      currentOwner = await ownerOf.call();
    } catch (e) {
      console.log("not minted already");
    }
    if (currentOwner) {
      this.$store.commit("updateOwner", {
        claim_status: true,
        currentOwner: currentOwner
      });
    } else {
      this.$store.commit("updateOwner", {
        claim_status: false,
        currentOwner: ""
      });
    }
  },
  methods: {
    connectMetamask: async function() {
      await this.ethereum.request({ method: "eth_requestAccounts" });
      // location.reload();
      this.$store.commit("change", {
        account: this.ethereum.selectedAddress,
        network: this.ethereum.networkVersion
      });
    },
    claim: async function() {
      this.$data.loading = true;

      // send generated image
      const image = this.$store.getters.tokenImage;
      await axios
        .post(apiUrl + "/gift/upload", {
          image: image,
          hash: this.$route.params.hash,
          token: this.$data.fromApi.token
        })
        .then(response => {
          console.log(response);
        });

      let mintFunction = contract.methods.mintGift(
        this.$store.getters.account,
        this.$data.fromApi.token.toString(),
        "0x" + this.$props.hash,
        this.$data.fromApi.proof
      );
      let ownerOfFunction = contract.methods.ownerOf(
        this.$data.fromApi.token.toString()
      );
      let currentOwner;
      try {
        this.$data.loading = true;
        currentOwner = await ownerOfFunction.call();
      } catch (err) {
        console.log("error in finding token / not already minted", err);
        this.$data.loading = false;
      }
      if (currentOwner) {
        this.$data.fielding.msg["alert"] = "owned by: " + currentOwner;
      } else {
        this.$data.loading = true;
        let data = await mintFunction.encodeABI(),
          gas;
        try {
          gas = await mintFunction.estimateGas();
        } catch (error) {
          console.log("you just made a boo boo", error);
          this.$data.loading = false;
        }
        if (gas) {
          this.$data.loading = true;
          const txParams = {
            gas: gas.toString(),
            to: contractNetwork.address,
            from: this.$store.getters.account,
            data
          };
          let transactionHash;
          try {
            transactionHash = await this.ethereum.request({
              method: "eth_sendTransaction",
              params: [txParams]
            });
          } catch (err) {
            console.log("err", err);
            this.$data.loading = false;
            this.$data.fielding.msg = "Request Rejected";
          }

          this.$data.fielding.msg["txHash"] = transactionHash;
          this.$data.linkTo.txHash = contractNetwork.txHashLink.replace(
            "<transaction>",
            transactionHash
          );
          // "https://blockscout.com/xdai/mainnet/tx/" +
          // transactionHash +
          // "/token-transfers";
          console.log(transactionHash);
        } else {
          this.$data.fielding.msg["alert"] = "Error in Gas Estimation";
        }
      }
      this.$data.loading = false;
    },
    timeout: function timeout(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    switchNetwork: async function() {
      console.log("clicked switch :)");
      let network = {
        chainId: "0x" + contractNetwork["chainId"].toString(16), // A 0x-prefixed hexadecimal string
        chainName: contractNetwork["name"],
        nativeCurrency: {
          name: contractNetwork["currency"],
          symbol: contractNetwork["currency"], // 2-6 characters long
          decimals: 18
        },
        rpcUrls: [contractNetwork["rpc"]],
        blockExplorerUrls: [contractNetwork["blockExplorerUrl"]]
      };
      let c = 0;
      c = await this.ethereum.request({
        method: "wallet_addEthereumChain",
        params: [network]
      });
      console.log(c);
      console.log("waiting 5 seconds...");
      await this.timeout(3000);
      if (c == null) {
        console.log("updating $store with:");
        console.log("account", this.ethereum.selectedAddress);
        console.log("network", this.ethereum.networkVersion);
        await this.$store.commit("change", {
          account: this.ethereum.selectedAddress,
          network: this.ethereum.networkVersion
        });
      }
    }
  }
};
</script>
