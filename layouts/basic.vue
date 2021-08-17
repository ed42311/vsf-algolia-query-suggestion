<template>
  <div id="basic" class="app-container">
    <h1>Autocomplete Test</h1>
    <QuerySuggestions />
    <AisInstantSearchSsr>
      <nuxt />
    </AisInstantSearchSsr>
  </div>
</template>

<script>
import { AisInstantSearchSsr, createServerRootMixin } from 'vue-instantsearch';
import QuerySuggestions from '../components/Header/QuerySuggestions.vue';
import algoliasearch from 'algoliasearch/lite';

const indexName = 'instant_search';
const searchClient = algoliasearch(
  'latency',
  '6be0576ff61c053d5f9a3225e2a90f76',
);

// const indexName = 'spryker_dev_murdochs_products';
// const searchClient = algoliasearch(
//   'H9ZUJZIP4J',
//   '0491e81cb21a934b9d37e559a19cb875',
// );

// remove indexName
// Props to: https://code.luasoftware.com/tutorials/algolia/setup-algolia-instantsearch-on-nuxt-with-query-url/
function writeState(routeState) {
  if (indexName in routeState) routeState = routeState[indexName];

  return routeState;
}

// restore indexName
// Props to: https://code.luasoftware.com/tutorials/algolia/setup-algolia-instantsearch-on-nuxt-with-query-url/
function readState(routeState) {
  routeState = {
    [indexName]: routeState,
  };
  return routeState;
}

function nuxtRouter(vueRouter) {
  return {
    read() {
      return readState(vueRouter.currentRoute.query);
    },
    write(routeState) {
      if (this.createURL(routeState) === this.createURL(this.read())) {
        return;
      }
      vueRouter.push({
        query: writeState(routeState),
      });
    },
    createURL(routeState) {
      return vueRouter.resolve({
        query: writeState(routeState),
      }).href;
    },
    onUpdate(cb) {
      if (typeof window === 'undefined') return;

      this._onPopState = event => {
        const routeState = event.state;
        if (!routeState) {
          cb(this.read());
        } else {
          cb(routeState);
        }
      };
      window.addEventListener('popstate', this._onPopState);
    },
    dispose() {
      if (typeof window === 'undefined') return;

      window.removeEventListener('popstate', this._onPopState);
    },
  };
}

export default {
  name: 'BasicLayout',
  serverPrefetch() {
    return this.instantsearch.findResultsState(this).then(algoliaState => {
      this.$ssrContext.nuxt.algoliaState = algoliaState;
    });
  },
  beforeMount() {
    const results =
      this.$nuxt.context.nuxtState.algoliaState || window.__NUXT__.algoliaState;
    this.instantsearch.hydrate(results);

    delete this.$nuxt.context.nuxtState.algoliaState;
    delete window.__NUXT__.algoliaState;
  },
  data() {
    const mixin = createServerRootMixin({
      searchClient,
      indexName,
      routing: {
        router: nuxtRouter(this.$router),
      },
    });
    return {
      ...mixin.data(),
    };
  },
  provide() {
    return {
      $_ais_ssrInstantSearchInstance: this.instantsearch,
    };
  },
  components: {
    QuerySuggestions,
    AisInstantSearchSsr,
  },
};
</script>

<style lang="scss">
@import '~@storefront-ui/vue/styles';

#basic {
  box-sizing: border-box;
  @include for-desktop {
    max-width: 1240px;
    margin: auto;
  }
}

// Reset CSS
html {
  width: auto;
  overflow-x: hidden;
}
body {
  overflow-x: hidden;
  color: var(--c-text);
  font-size: var(--font-size--base);
  font-family: var(--font-family--primary);
  margin: 0;
  padding: 0;
}
a {
  text-decoration: none;
  color: var(--c-link);
  &:hover {
    color: var(--c-link-hover);
  }
}
h1 {
  font-family: var(--font-family--secondary);
  font-size: var(--h1-font-size);
  line-height: 1.6;
  margin: 0;
}
h2 {
  font-family: var(--font-family--secondary);
  font-size: var(--h2-font-size);
  line-height: 1.6;
  margin: 0;
}
h3 {
  font-family: var(--font-family--secondary);
  font-size: var(--h3-font-size);
  line-height: 1.6;
  margin: 0;
}
h4 {
  font-family: var(--font-family--secondary);
  font-size: var(--h4-font-size);
  line-height: 1.6;
  margin: 0;
}
</style>
