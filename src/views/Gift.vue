<template>
  <div>
    <div
      class="min-h-screen min-w-screen py-6 flex flex-col justify-center sm:py-12"
    >
      <div class="relative py-3 sm:max-w-xl sm:mx-auto">
        <div
          class="absolute inset-0 bg-gradient-to-r from-purple-500 to-purple-900 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"
        ></div>
        <div
          class="relative px-1 py-10 bg-gray-50 shadow-lg sm:rounded-3xl sm:p-20"
        >
          <div
            class="invisible sm:visible absolute sm:bottom-0 sm:left-0 sm:p-5 font-fancy text-gray-400 text-lg"
            v-if="!fielding.loading"
          >
            fancy a
            <span class="tweet-link">
              <a :href="tweetText" target="_new" class="text-blue-500"
                >tweet <font-awesome-icon :icon="['fab', 'twitter']"
              /></a>
              ?
            </span>
          </div>
          <div
            class="invisible sm:visible sm:absolute sm:bottom-0 sm:bottom-0 sm:right-0 sm:p-5"
            v-if="!fielding.loading"
          >
            <span class="mr-1 font-fancy">
              <span
                class="border-solid border-b-2 border-gray-600 cursor-pointer"
                @click="$store.commit('togglePanel', true)"
              >
                View Testimonials
              </span>
            </span>
          </div>
          <div class="absolute bottom-0 p-3 sm:invisible">
            <span class="tweet-link">
              <a :href="tweetText" target="_new" class="text-blue-500"
                >Tweet <font-awesome-icon :icon="['fab', 'twitter']"
              /></a>
            </span>
            |
            <span
              class="border-solid border-b-2 border-gray-600 cursor-pointer"
              @click="$store.commit('togglePanel', true)"
            >
              View Testimonials
            </span>
          </div>
          <div class="max-w-md mx-auto">
            <div v-if="!fielding.loading">
              <div v-if="fielding.ok">
                <p class="font-heading font-bold text-4xl sm:text-5xl">
                  {{ fromApi.name }}
                </p>
                <div v-if="network != 'mainnet'" class="text-xs text-gray-400">
                  <span>
                    network:
                    <span class="font-mono text-xs">
                      {{ network }}
                    </span>
                  </span>
                </div>
                <div class="text-gray-400 text-xs">
                  token id:
                  {{ fromApi.token }}
                </div>
                <!-- <p class="font-heading text-5xl">
              Hello!
            </p> -->
                <!-- <p class="truncate font-mono text-xs">
              {{ hash }}
            </p> -->
                <!-- <p>loading: {{ fielding.loading }}</p> -->
                <div>
                  <!-- <p>claim: {{ fromApi.claim_status }}</p> -->
                  <!-- <p>proof: {{ fromApi.proof }}</p> -->
                  <p
                    v-if="Object.keys(fromApi.award).length > 0"
                    class="font-award text-4xl mt-5"
                  >
                    {{ fromApi.award }}
                  </p>
                  <!-- <p v-if="fromApi.gratitude">gratitude: {{ fromApi.gratitude }}</p> -->
                  <!-- <p>image: {{ fromApi.image }}</p> -->
                  <!-- insert image here -->
                  <!-- <div id="canvas"></div> -->
                  <div class="my-5">
                    <Canvas
                      :fellowName="fromApi.name"
                      class="h-96 w-96 shadow-2xl"
                    />
                  </div>
                </div>
                <div v-if="!loading">
                  <!-- if claim status = false / not already minted -->
                  <div v-if="!$store.getters.claim_status">
                    <!-- display if metamask is not connected -->
                    <div v-if="!$store.getters.account">
                      <button
                        class="bg-purple-500 hover:bg-purple-900 text-white font-bold py-3 px-4 rounded-lg shadow-2xl font-kernel"
                        v-on:click="connectMetamask"
                      >
                        Connect Metamask
                      </button>
                    </div>
                    <!-- display if metamask is connected -->
                    <div v-else>
                      <!-- display if the metamask is connected to the correct network -->
                      <!-- <button
                        class="bg-purple-500 hover:bg-purple-900 text-white font-bold py-3 px-4 mr-2 rounded-lg shadow-2xl font-kernel"
                        v-on:click="generate"
                        v-if="$store.getters.network == $store.getters.chainId"
                      >
                        Generate NFT
                      </button> -->
                      <button
                        class="bg-purple-500 hover:bg-purple-900 text-white font-bold mb-4 py-3 px-4 rounded-lg shadow-2xl font-kernel"
                        v-on:click="claim"
                        v-if="$store.getters.network == $store.getters.chainId"
                      >
                        Claim
                      </button>

                      <!-- display if the metamask is not connected to the correct network -->
                      <div v-else>
                        <p>
                          <!-- Please connect to chain id
                        {{ $store.getters.chainId }} & reload -->
                          <a
                            class="cursor-pointer"
                            v-on:click="switchNetwork"
                            v-if="network == 'matic' || network == 'mumbai'"
                            >Click here to connect</a
                          >
                          <span v-else>Switch</span>
                          to {{ network }} network. Reload your window after you
                          have switched.
                        </p>
                      </div>
                    </div>
                  </div>
                  <!-- if already minted -->
                  <div v-else>
                    <p class="truncate">
                      The token is minted to:
                      <span class="font-mono truncate">
                        <a :href="linkTo.tokens" target="_new">
                          {{ $store.getters.currentOwner }}
                        </a>
                      </span>
                    </p>
                  </div>
                </div>
                <div v-else>
                  <div class="spinner"></div>
                </div>
              </div>

              <div v-else>
                <p class="py-3 text-2xl">{{ fielding.notAuthMsg }}</p>
              </div>

              <div v-if="fielding.msg.txHash" class="pt-4">
                <p>
                  Minting transaction sent
                </p>
                <p class="truncate">
                  Transaction hash:
                  <span class="font-mono truncate">
                    <a :href="linkTo.txHash" target="_new">
                      {{ fielding.msg.txHash }}
                    </a>
                  </span>
                </p>
              </div>
              <div v-if="fielding.msg.alert" class="pt-4">
                <p>
                  {{ fielding.msg.alert }}
                </p>
              </div>
            </div>
            <div v-else>
              <div class="spinner"></div>
            </div>
          </div>
        </div>
      </div>
      <div class="static page-footer bottom-0 left-0">
        <div
          class="absolute md:bottom-0 md:left-0 p-5 mt-6 font-sans text-gray-400 text-sm"
        >
          <span>
            Design & code by
          </span>
          <a href="https://www.twitter.com/angelagilhotra/" target="_new">
            @angelagilhotra
          </a>
          <span>
            , special thanks to
          </span>
          <a href="https://www.instagram.com/malayvasa/" target="_new">
            @malayvasa
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Canvas from "@/components/Canvas";
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
    Canvas
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
  metaInfo() {
    return {
      title: `, ${this.fromApi.name}`,
      meta: [
        {
          name: "description",
          content:
            "KERNEL Gratitude is a token of appreciation for each fellow in the network, as unique as their contribution. It's a living artifact of a fellow's presence in the community's virtual abode 🏡."
        },
        { property: "og:title", content: `Thank You, ${this.fromApi.name}` },
        {
          property: "og:site_name",
          content: `Thank You, ${this.fromApi.name}`
        },
        {
          property: "og:description",
          content:
            "KERNEL Gratitude is a token of appreciation for each fellow in the network, as unique as their contribution. It's a living artifact of a fellow's presence in the community's virtual abode 🏡."
        },
        { property: "og:type", content: "page" },
        {
          property: "og:url",
          content: "https://gratitude.kernel.community/c/" + this.props.hash
        },
        {
          property: "og:image",
          content: "https://api.kernel.community/" + this.fromApi.token + ".png"
        }
      ]
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
