<template>
  <div>
    <div id="autocomplete" />
  </div>
</template>

<script>
import { h } from 'vue';
import { onMounted, render, Fragment } from '@vue/composition-api';
import algoliasearch from 'algoliasearch/lite';
import { autocomplete, getAlgoliaResults } from '@algolia/autocomplete-js';

export default {
  name: 'Search',
  layout(context) {
    return 'test';
  },
  setup() {
    onMounted(() => {
      autocomplete({
        container: '#autocomplete',
        openOnFocus: true,
        getSources({ query }) {
          return [
            {
              sourceId: 'products',
              getItems() {
                return getAlgoliaResults({
                  searchClient,
                  queries: [
                    {
                      indexName: 'instant_search',
                      query,
                      params: {
                        hitsPerPage: 10,
                        attributesToSnippet: ['name:10', 'description:35'],
                        snippetEllipsisText: '…',
                      },
                    },
                  ],
                });
              },
              templates: {
                item({ item, components }) {
                  return (
                    <div className="aa-ItemWrapper">
                      <div className="aa-ItemContent">
                        <div className="aa-ItemIcon">
                          <img
                            src={hit.image}
                            alt={hit.name}
                            width="40"
                            height="40"
                          />
                        </div>
                        <div className="aa-ItemContentBody">
                          <div className="aa-ItemContentTitle">
                            <components.Snippet hit={item} attribute="name" />
                          </div>
                          <div className="aa-ItemContentDescription">
                            <components.Snippet
                              hit={item}
                              attribute="description"
                            />
                          </div>
                        </div>
                      </div>
                    </div>
                  );
                },
              },
            },
          ];
        },
        renderer: {
          createElement: h,
          Fragment,
        },
        render({ children }, root) {
          render(children, root);
        },
      });
    });
  },
};
</script>
