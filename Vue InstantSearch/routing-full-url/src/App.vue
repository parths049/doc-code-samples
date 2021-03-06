<template>
  <div>
    <header class="header">
      <h1 class="header-title"><a href="/">Vue InstantSearch v2 starter</a></h1>
      <p class="header-subtitle">
        using
        <a href="https://github.com/algolia/vue-instantsearch">
          Vue InstantSearch
        </a>
      </p>
    </header>

    <div class="container">
      <ais-instant-search
        :search-client="searchClient"
        index-name="instant_search"
        :routing="routing"
      >
        <div class="search-panel">
          <div class="search-panel__filters">
            <ais-refinement-list attribute="brand" searchable />
          </div>

          <div class="search-panel__results">
            <ais-search-box placeholder="Search here…" class="searchbox" />
            <ais-hits>
              <template slot="item" slot-scope="{ item }">
                <h1><ais-highlight :hit="item" attribute="name" /></h1>
                <p><ais-highlight :hit="item" attribute="description" /></p>
              </template>
            </ais-hits>

            <div class="pagination"><ais-pagination /></div>
          </div>
        </div>
      </ais-instant-search>
    </div>
  </div>
</template>

<script>
import algoliasearch from 'algoliasearch/lite';
import { history as historyRouter } from 'instantsearch.js/es/lib/routers';
import 'instantsearch.css/themes/algolia-min.css';

export default {
  data() {
    return {
      searchClient: algoliasearch(
        'latency',
        '6be0576ff61c053d5f9a3225e2a90f76'
      ),
      routing: {
        router: historyRouter({
          windowTitle(routeState) {
            return `Website / Find ${routeState.q} in ${
              routeState.brands
            } brands`;
          },
          createURL({ routeState, location }) {
            let baseUrl = location.href.split('/search/')[0];
            if (
              !routeState.q &&
              routeState.brands === 'all' &&
              routeState.p === 1
            )
              return baseUrl;
            if (baseUrl[baseUrl.length - 1] !== '/') baseUrl += '/';
            let routeStateArray = [
              'q',
              encodeURIComponent(routeState.q),
              'brands',
              encodeURIComponent(routeState.brands),
              'p',
              routeState.p,
            ];

            return `${baseUrl}search/${routeStateArray.join('/')}`;
          },
          parseURL({ location }) {
            let routeStateString = location.href.split('/search/')[1];
            if (routeStateString === undefined) return {};
            const routeStateValues = routeStateString.match(
              /^q\/(.*?)\/brands\/(.*?)\/p\/(.*?)$/
            );
            return {
              q: decodeURIComponent(routeStateValues[1]),
              brands: decodeURIComponent(routeStateValues[2]),
              p: routeStateValues[3],
            };
          },
        }),
        stateMapping: {
          stateToRoute(uiState) {
            return {
              q: uiState.query || '',
              brands:
                (uiState.refinementList &&
                  uiState.refinementList.brand &&
                  uiState.refinementList.brand.join('~')) ||
                'all',
              p: uiState.page || 1,
            };
          },
          routeToState(routeState) {
            if (routeState.brands === 'all') routeState.brands = undefined;

            return {
              query: routeState.q,
              refinementList: {
                brand: routeState.brands && routeState.brands.split('~'),
              },
              page: routeState.p,
            };
          },
        },
      },
    };
  },
};
</script>

<style>
body,
h1 {
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica,
    Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol';
}

.ais-Highlight-highlighted {
  background: cyan;
  font-style: normal;
}

.header {
  display: flex;
  align-items: center;
  min-height: 50px;
  padding: 0.5rem 1rem;
  background-image: linear-gradient(to right, #4dba87, #2f9088);
  color: #fff;
  margin-bottom: 1rem;
}

.header a {
  color: #fff;
  text-decoration: none;
}

.header-title {
  font-size: 1.2rem;
  font-weight: normal;
}

.header-title::after {
  content: ' ▸ ';
  padding: 0 0.5rem;
}

.header-subtitle {
  font-size: 1.2rem;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
}

.search-panel {
  display: flex;
}

.search-panel__filters {
  flex: 1;
  margin-right: 1em;
}

.search-panel__results {
  flex: 3;
}

.searchbox {
  margin-bottom: 2rem;
}

.pagination {
  margin: 2rem auto;
  text-align: center;
}
</style>
