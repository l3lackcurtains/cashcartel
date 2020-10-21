<template>
  <div>
    <section class="hero is-medium is-light">
      <div class="hero-body">
        <div class="container">
          <div class="columns is-vcentered">
            <div class="column is-4">
              <div class="title">CASH CARTEL</div>
              <div class="subtitle">TOP 100 MEMBERS</div>
            </div>

            <div class="column">
              <div class="columns is-multiline">
                <b-table :data="data" :per-page="10" paginated v-if="loadtable">
                  <b-table-column
                    field="address"
                    label="Address"
                    v-slot="props"
                  >
                    {{ props.row.address }}
                  </b-table-column>
                  <b-table-column field="share" label="Share" v-slot="props">
                    {{ props.row.share }}
                  </b-table-column>
                  <b-table-column field="address" label="Zerion" v-slot="props">
                    <a
                      target="_blank"
                      v-bind:href="
                        'https://app.zerion.io/' +
                        props.row.address +
                        '/overview'
                      "
                    >
                      <b-button size="is-small"> Zerion</b-button>
                    </a>
                  </b-table-column>
                  <b-table-column
                    field="address"
                    label="Uniswap"
                    v-slot="props"
                  >
                    <a
                      target="_blank"
                      v-bind:href="
                        'https://info.uniswap.org/account/' + props.row.address
                      "
                    >
                      <b-button size="is-small">Uniswap</b-button>
                    </a>
                  </b-table-column>

                  <b-table-column field="address" label="Twitter" v-slot="props">
                    <div v-if="props.row.twitter">
                      {{ props.row.twitter }}
                    </div>
                    <div v-else>
                      <b-button size="is-small" type="is-link is-light" @click="isCardModalActive = true">Set Twitter</b-button>
                    </div>
                  </b-table-column>
                </b-table>

                <b-modal v-model="isCardModalActive" :width="400">
                  <div class="card">
                        <section class="content">
                          <b-field label="Enter twitter username">
                                <b-input name="twitter" expanded v-model="twitterAccount"></b-input>
                            </b-field>
                            <b-field>
                              <p class="control">
                                  <button class="button is-primary" @click="setTwitter">
                                    Set Twitter
                                  </button>
                              </p>
                          </b-field>
                          </section>
                    </div>
              </b-modal>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>


<script>
import axios from "axios";




import Web3 from 'web3';

const abi = require("../utils/abi.json");

window.web3 = new Web3(new Web3.providers.HttpProvider("https://mainnet.infura.io/v3/1d106c3131ba40c898b399c3b52d78ed"));

if (window.ethereum) {
    window.web3 = new Web3(window.ethereum);
    window.ethereum.enable();
}

const contractAddress = "0x9257E8c80119fEDAB64842024ced4113912B0aB1";

const casinoContract = new window.web3.eth.Contract(abi, contractAddress);

export default {
  data() {
    return {
      data: "",
      loadtable: false,
      isCardModalActive: false,
      twitterAccount: ""
    };
  },

  mounted() {
    const newLocal = this;

    axios
      .get(
        "https://api.ethplorer.io/getTopTokenHolders/0x9257E8c80119fEDAB64842024ced4113912B0aB1?apiKey=freekey&limit=100"
      )
      .then(async (response) => {
        let holdersInfo = response.data.holders;

        try {
          holdersInfo =  await Promise.all(holdersInfo.map(async holder => {
            const twitter = await casinoContract.methods.getTwitter(holder.address).call();
            holder.twitter = twitter;
            return holder;
          }));
        } catch(e) {
          console.log(e);
        }

        newLocal.data = holdersInfo;

        newLocal.loadtable = true;
      })
      .catch(function (error) {
        console.log(error);
      });
      
  },

  methods: {
    setTwitter: function() {

      if (window.ethereum) {
          window.web3 = new Web3(window.ethereum);
          window.ethereum.enable();
      }

      window.web3.eth.getAccounts().then(accounts => {
        casinoContract.methods.setTwitter(this.twitterAccount).send({ from: accounts[0]});
      });
    }
  }
};
</script>

<style>
.card .content {
  padding: 36px;
}
</style>


