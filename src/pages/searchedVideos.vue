<template>
  <div>
    <scroller lock-x height="-45"
        :pulldown-config="pulldownConfig" 
        :pullup-config="pullupConfig" ref="theScroller" 
        :use-pulldown=true :use-pullup=true 
        @on-pulldown-loading="refresh()" 
        @on-pullup-loading="loadMore()">
      <div>
        <UploadedItem mode="search" :video="video" :key="video.id" v-for="video in videolist" />
        <div v-if="videolist.length == 0" style="text-align: center"><br>暂无搜索结果</div>
      </div>
    </scroller>
  </div>
</template>

<script>
import { Scroller, LoadMore } from "vux";
import UploadedItem from "../components/item/uploadedItem.vue";
export default {
  components: {
    Scroller,
    LoadMore,
    UploadedItem
  },
  data() {
    return {
      videolist: [],
      psize: 10,
      pulldownConfig: {
        downContent: "下拉刷新",
        upContent: "释放后更新"
      },
      pullupConfig: {
        upContent: "",
        downContent: "释放后加载"
      }
    };
  },
  mounted() {
    this.searchVideos();
  },
  methods: {
    searchVideos() {
      var self = this;
      this.$axios
        .get("/videos/all/found", {
          params: {
            title: this.$route.query.title,
            pstart: this.videolist.length,
            psize: this.psize
          }
        })
        .then(function(response) {
          self.videolist.push.apply(self.videolist, response.data);
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    refresh() {
      this.videolist = [];
      this.searchVideos();
      this.$nextTick(() => {
        this.$refs.theScroller.donePulldown();
        this.$refs.theScroller.reset({ top: 0 });
      });
    },
    loadMore() {
      this.searchVideos();
      this.$nextTick(() => {
        this.$refs.theScroller.donePullup();
        this.$refs.theScroller.reset();
      });
    }
  }
};
</script>