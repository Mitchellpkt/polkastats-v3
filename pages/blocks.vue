<template>
  <div>
    <section>
      <b-container class="main py-5">
        <h1 class="mb-4">
          {{ $t("pages.blocks.title") }}
        </h1>
        <div class="last-blocks">
          <div class="table-responsive">
            <b-table striped hover :fields="fields" :items="blocks">
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
              <template v-slot:cell(block_hash)="data">
                <p class="mb-0">
                  {{ shortHash(data.item.block_hash) }}
                </p>
              </template>
              <template v-slot:cell(block_author)="data">
                <p class="mb-0 d-inline-block">
                  <Identicon
                    :key="data.item.block_author"
                    :value="data.item.block_author"
                    :size="20"
                    :theme="'polkadot'"
                  />
                  <nuxt-link
                    v-b-tooltip.hover
                    :to="`/validator?accountId=${data.item.block_author}`"
                    title="Check validator information"
                  >
                    <span v-if="data.item.block_author_name">
                      {{ data.item.block_author_name }}
                    </span>
                    <span v-else>
                      {{ shortAddress(data.item.block_author) }}
                    </span>
                  </nuxt-link>
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
      blocks: [],
      fields: [
        {
          key: "block_number",
          label: "Block",
          sortable: true
        },
        {
          key: "block_hash",
          label: "Hash",
          sortable: true
        },
        {
          key: "block_author",
          label: "Author",
          sortable: true
        }
      ]
    };
  },
  apollo: {
    $subscribe: {
      block: {
        query: gql`
          subscription blocks {
            block(order_by: { block_number: desc }, where: {}, limit: 50) {
              block_number
              block_hash
              block_author
              block_author_name
              current_era
              current_index
              era_length
              era_progress
              extrinsics_root
              is_epoch
              new_accounts
              num_transfers
              parent_hash
              session_length
              session_per_era
              session_progress
              spec_name
              spec_version
              state_root
              timestamp
              total_events
              validator_count
            }
          }
        `,
        result({ data }) {
          this.blocks = data.block;
        }
      }
    }
  },
  head() {
    return {
      title: this.$t("pages.blocks.head_title", {
        networkName: network.name
      }),
      meta: [
        {
          hid: "description",
          name: "description",
          content: this.$t("pages.blocks.head_content", {
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
