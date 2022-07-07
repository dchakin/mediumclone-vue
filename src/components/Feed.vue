<template>
  <div>
    <Loading v-if='isLoading' />
    <ErrorMessage v-if='error' />

    <div v-if='feed'>
      <div class='article-preview' v-for='(article, index) in feed.articles' :key='index'>
        <div class='article-meta'>
          <router-link to="{name: 'userProfile', params: {slug: article.author.userName}}">
            <img :src='article.author.image'>
          </router-link>
          <div class='info'>
            <router-link to="{name: 'userProfile', params: {slug: article.author.userName}}"
                         class='author'
            >
              {{ article.author.username }}
            </router-link>
            <span class='date'>
              {{ article.createdAt }}
            </span>
          </div>
          <div class='pull-xs-right'>
            ADD TO FAVOURITES
          </div>
        </div>
        <router-link to='{name: "article", params: {slug: article.slug}}' class='preview-link'>
          <h1>{{ article.title }}</h1>
          <p>{{ article.description }}</p>
          <span>Read more...</span>
          TAG LIST
        </router-link>
      </div>
      <Pagination
        :total='total'
        :limit='limit'
        :current-page='currentPage'
        :url='baseUrl'
      />
    </div>
  </div>
</template>

<script>
import {mapState} from 'vuex';
import {actionTypes} from '@/store/modules/feed';
import Pagination from '@/components/Pagintaion';
import {limit} from '@/helpers/vars';
import {stringify, parseUrl} from 'query-string';
import Loading from '@/components/Loading';
import ErrorMessage from '@/components/ErrorMessage';

export default {
  name: 'Feed',
  components: {
    Pagination,
    Loading,
    ErrorMessage
  },
  data() {
    return {
      total: 100,
      limit,
      url: '/'
    };
  },
  props: {
    apiUrl: {
      type: String,
      required: true
    }
  },
  computed: {
    ...mapState({
      isLoading: state => state.feed.isLoading,
      feed: state => state.feed.data,
      error: state => state.feed.error
    }),
    currentPage() {
      return Number(this.$route.query.page || '1');
    },
    baseUrl() {
      return this.$route.path;
    },
    offset() {
      return this.currentPage * limit - limit;
    }
  },
  watch: {
    currentPage() {
      this.fetchFeed();
    }
  },
  mounted() {
    this.fetchFeed();
  },
  methods: {
    fetchFeed() {
      const parsedUrl = parseUrl(this.apiUrl);
      const stringifiedParams = stringify({
        limit,
        offset: this.offset,
        ...parsedUrl.query
      });
      const apiUrlWithParams = `${parsedUrl.url}?${stringifiedParams}`;
      this.$store.dispatch(actionTypes.getFeed, {apiUrl: apiUrlWithParams});
    }
  }

};
</script>

<style scoped>

</style>
