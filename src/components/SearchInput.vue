<template>
  <div class="relative">
    <search-focus @keyup="focusSearch"></search-focus>

    <div class="relative w-80">
    </div>
    <transition name="fade">
      <div v-if="query.length > 0 && searchResultsVisible" class="normal-case absolute border left-0 right-0 w-108 text-left mb-4 mt-2 rounded-lg shadow overflow-hidden z-10 overflow-y-auto" style="max-height: 32rem" data-cypress="search-results">
        <div class="flex flex-col" ref="results">
          <a
            v-for="(post, index) in results"
            :key="index"
            :href="post.item.path"
            @click="reset"
            class="bg-background-form border-b border-gray-400 text-xl cursor-pointer p-4 search-hover"
            :class="{ 'search-highlighted' : index === highlightedIndex }"
          >
            {{ post.item.title }}

            <span class="block font-normal text-copy-primary text-sm my-1">{{ post.item.summary }}</span>
          </a>

          <div v-if="results.length === 0" class="bg-background-form font-normal w-full border-b cursor-pointer p-4">
            <p class="my-0">No results for '<strong>{{ query }}</strong>'</p>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>


<static-query>
query Search {
  allPost {
    edges {
      node {
        id
        path
        title
        summary
        headings {
          depth
          value
          anchor
        }
      }
    }
  }
  allDocumentation {
    edges {
      node {
        id
        path
        title
      }
    }
  }
}
</static-query>

<script>
import SearchFocus from './SearchFocus'

export default {
  components: {
    SearchFocus,
  },
  computed: {
    pages () {
      let result = [];
      const allPost = this.$static.allPost.edges.map(edge => edge.node);
      allPost.forEach(page => {
        result.push({
          path: page.path,
          title: page.title,
          summary: page.summary
        });
      });
      const allDocs = this.$static.allDocumentation.edges.map(edge => edge.node);
      allDocs.forEach(page => {
        result.push({
          path: page.path,
          title: page.title
        });
      });
      return result;
    }
  },
  data() {
    return {
      query: '',
      results: [],
      highlightedIndex: 0,
      searchResultsVisible: false,
      options: {
        shouldSort: true,
        includeMatches: true,
        threshold: 0.5,
        location: 0,
        distance: 500,
        maxPatternLength: 32,
        minMatchCharLength: 1,
        keys: ['title', 'summary']
      }
    }
  },
  methods: {
    reset() {
      this.query = ''
      this.highlightedIndex = 0
    },
    softReset() {
      this.highlightedIndex = 0
      this.searchResultsVisible = true
    },
    performSearch() {
      this.$search(this.query, this.pages, this.options).then(results => {
        this.results = results
      })
    },
    highlightPrev() {
      if (this.highlightedIndex > 0) {
        this.highlightedIndex = this.highlightedIndex - 1
        this.scrollIntoView()
      }
    },
    highlightNext() {
      if (this.highlightedIndex < this.results.length - 1) {
        this.highlightedIndex = this.highlightedIndex + 1
        this.scrollIntoView()
      }
    },
    scrollIntoView() {
      this.$refs.results.children[this.highlightedIndex].scrollIntoView({ block: 'nearest' })
    },
    gotoLink() {
      if (this.results[this.highlightedIndex]) {
        window.location = this.results[this.highlightedIndex].item.path
      }
    },
    focusSearch(e) {
      if (e.key === '/') {
        this.$refs.search.focus()
      }
    }
  }
}
</script>

<style scoped>
  .fade-enter-active, .fade-leave-active {
    transition: opacity .2s;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>

