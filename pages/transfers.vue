<template>
  <div>
    <section>
      <b-container class="main py-5">
        <h1 class="mb-4">
          {{ $t("pages.transfers.title") }}
        </h1>
        <div class="last-transfers">
          <div class="table-responsive">
            <b-table striped hover :fields="fields" :items="transfers">
              <template v-slot:cell(block_number)="data">
                <p class="mb-0">
                  <nuxt-link
                    v-b-tooltip.hover
                    :to="`/block?blockNumber=${data.item.block_number}`"
                    title="Check block information"
                  >
                    #{{ formatNumber(data.item.block_number) }}
                  </nuxt-link>
                </p>
              </template>
              <template v-slot:cell(hash)="data">
                <p class="mb-0">
                  {{ shortHash(data.item.hash) }}
                </p>
              </template>
              <template v-slot:cell(from)="data">
                <p class="mb-0">
                  <nuxt-link
                    :to="{
                      name: 'account',
                      query: { accountId: data.item.from }
                    }"
                    :title="$t('pages.accounts.account_details')"
                  >
                    <Identicon
                      :key="data.item.from"
                      :value="data.item.from"
                      :size="20"
                      :theme="'polkadot'"
                    />
                    <span v-if="getDisplayName(data.item.from)">
                      {{ getDisplayName(data.item.from) }}
                    </span>
                    <span v-else>
                      {{ shortAddress(data.item.from) }}
                    </span>
                  </nuxt-link>
                </p>
              </template>
              <template v-slot:cell(to)="data">
                <p class="mb-0">
                  <nuxt-link
                    :to="{
                      name: 'account',
                      query: { accountId: data.item.to }
                    }"
                    :title="$t('pages.accounts.account_details')"
                  >
                    <Identicon
                      :key="data.item.to"
                      :value="data.item.to"
                      :size="20"
                      :theme="'polkadot'"
                    />
                    <span v-if="getDisplayName(data.item.to)">
                      {{ getDisplayName(data.item.to) }}
                    </span>
                    <span v-else>
                      {{ shortAddress(data.item.to) }}
                    </span>
                  </nuxt-link>
                </p>
              </template>
              <template v-slot:cell(amount)="data">
                <p class="mb-0">
                  {{ formatAmount(data.item.amount) }}
                </p>
              </template>
              <template v-slot:cell(success)="data">
                <p class="mb-0">
                  <i
                    v-if="data.item.success"
                    class="fa fa-check-circle text-success"
                    aria-hidden="true"
                  ></i>
                  <i
                    v-else
                    class="fa fa-check-circle text-danger"
                    aria-hidden="true"
                  ></i>
                </p>
              </template>
            </b-table>
          </div>
        </div>
      </b-container>
    </section>
  </div>
</template>

<script>
import commonMixin from "../mixins/commonMixin.js";
import Identicon from "../components/identicon.vue";
import gql from "graphql-tag";
import { network } from "../polkastats.config.js";

export default {
  components: {
    Identicon
  },
  mixins: [commonMixin],
  data: function() {
    return {
      transfers: [],
      fields: [
        {
          key: "block_number",
          label: "Id",
          sortable: true
        },
        {
          key: "hash",
          label: "Hash",
          sortable: true
        },
        {
          key: "from",
          label: "From",
          sortable: true
        },
        {
          key: "to",
          label: "To",
          sortable: true
        },
        {
          key: "amount",
          label: "Amount",
          sortable: true
        },
        {
          key: "success",
          label: "Success",
          sortable: true
        }
      ]
    };
  },
  created: function() {
    var vm = this;
    // Force update of identity list if empty
    if (this.$store.state.identities.list.length === 0) {
      vm.$store.dispatch("identities/update");
    }
    // Update data every 60 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch("identities/update");
    }, 60000);
  },
  methods: {
    getDisplayName: function(accountId) {
      let identity = this.$store.state.identities.list.find(
        identity => identity.accountId === accountId
      );
      if (identity) {
        identity = identity.identity;
        if (
          identity.displayParent &&
          identity.displayParent !== `` &&
          identity.display &&
          identity.display !== ``
        ) {
          return `${identity.displayParent} / ${identity.display}`;
        } else {
          return identity.display;
        }
      }
      return ``;
    }
  },
  apollo: {
    $subscribe: {
      extrinsic: {
        query: gql`
          subscription extrinsic {
            extrinsic(
              order_by: { block_number: desc }
              where: {
                section: { _eq: "balances" }
                method: { _eq: "transfer" }
              }
              limit: 50
            ) {
              block_number
              signer
              hash
              args
              success
            }
          }
        `,
        result({ data }) {
          this.transfers = data.extrinsic.map(transfer => {
            return {
              block_number: transfer.block_number,
              hash: transfer.hash,
              from: transfer.signer,
              to: JSON.parse(transfer.args)[0],
              amount: JSON.parse(transfer.args)[1],
              success: transfer.success
            };
          });
        }
      }
    }
  },
  head() {
    return {
      title: this.$t("pages.transfers.head_title", {
        networkName: network.name
      }),
      meta: [
        {
          hid: "description",
          name: "description",
          content: this.$t("pages.transfers.head_content", {
            networkName: network.name
          })
        }
      ]
    };
  }
};
</script>

<style>
.last-blocks .identicon {
  display: inline-block;
  margin: 0 0.2rem 0 0;
  cursor: copy;
}
</style>
