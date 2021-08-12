<template>
  <AisAutocomplete>
    <div slot-scope="{ currentRefinement, indices, refine }">
      <input
        type="search"
        :value="currentRefinement"
        placeholder="Search for a product"
        @input="refine($event.currentTarget.value)"
      />
      <div v-if="currentRefinement">
        <ul v-for="index in indices" :key="index.indexId">
          <li>
            <h3>{{ index.indexName }}</h3>
            <ul>
              <li v-for="hit in index.hits" :key="hit.objectID">
                <ais-highlight attribute="name" :hit="hit" />
                <button
                  type="button"
                  @click="index.sendEvent('click', hit, 'Item Added')"
                >
                  Add to cart
                </button>
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
  </AisAutocomplete>
</template>

<script>
import { AisAutocomplete } from 'vue-instantsearch';
export default {
  name: 'QuerySuggestions',
  setup(_, context) {
    console.log('query');
  },
  components: {
    AisAutocomplete,
  },
};
</script>

<style lang="scss"></style>
