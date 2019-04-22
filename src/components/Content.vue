<template>
  <div>
    <el-container class="container">

      <el-header height="200px">
        <h3>Github Repo URL</h3>
        <el-form class="search-form" @submit.native.prevent>
          <el-form-item>
            <el-input autofocus v-model="repoURL" @keyup.enter.native="startAnalysis" :placeholder="defaultURL"></el-input>
          </el-form-item>

          <el-form-item>
            <el-button type="info" @click="startAnalysis" plain><i :class="beginStatusClass"></i>Begin Anlysis</el-button>
          </el-form-item>
        </el-form>
      </el-header>

      <el-main v-if="data.length !== 0">
        <!-- top total downloads -->
        <div class="common-info-container" v-if="totalDownloads">
          <el-alert
            :closable="false"
            type="info"
            id="top-total"
            >
            <i class="el-icon-download">Total Downloads: <b>{{totalDownloads}}</b></i>
          </el-alert>

          <el-card>
            <el-table v-if="downloadsByReleases.length != 0"
              :data="downloadsByReleases"
              show-summary
              sum-text="Total Downloads"
              max-height="300"
              >
              <el-table-column
                prop="name"
                label="Release"
                >
                  <template slot-scope="scope">
                    <i class="el-icon-goods"></i>
                    <a style="margin-left: 10px">{{ scope.row.tag }}</a>
                  </template>
              </el-table-column>
              <el-table-column
                sortable
                prop="count"
                label="Download"
                >
              </el-table-column>
            </el-table>
          </el-card>

        </div>

        <!-- release cards -->
        <el-card v-for="release in data" class="release-cards">
          <div slot="header" class="clearfix">
            <span class="el-icon-star-on release-card-tag-name">{{ release.tag_name}}</span>
            <a :href="release.html_url">{{ release.name}}</a>
            <span class="release-card-right el-icon-time"> {{ new Date(release.published_at).toLocaleDateString() }} by <a :href="'https://github.com/' + release.author.login">{{ release.author.login }}</a></span>
          </div>

          <div>
            <el-table v-if="release.assets && release.assets.length != 0"
              :data="release.assets"
              :show-summary="release.assets.length > 1"
              sum-text="Total Downloads"
              >
              <el-table-column
                prop="name"
                label="Assets"
                >
                  <template slot-scope="scope">
                    <i class="el-icon-sold-out"></i>
                    <a :href="scope.row.browser_download_url" style="margin-left: 10px">{{ scope.row.name }}</a>
                  </template>
              </el-table-column>
              <el-table-column
                prop="download_count"
                label="Download"
                >
              </el-table-column>
            </el-table>
          </div>
        </el-card>
      </el-main>
    </el-container>

    <div class="copyright-declare">Powered by <a href="https://qii404.me">qii404.me</a></div>
  </div>
</template>

<script type="text/javascript">
  import gh from 'parse-github-url';

  export default {
    data() {
      return {
        repoURL: '',
        data: [],
        beginStatusClass: '',
        defaultURL: 'https://github.com/qishibo/AnotherRedisDesktopManager/',
      };
    },
    computed: {
      totalDownloads() {
        let count = 0;

        for (const release of this.data) {
          for (const asset of release.assets) {
            count += asset.download_count;
          }
        }

        return count;
      },
      downloadsByReleases() {
        let download = [];

        for (const release of this.data) {
          let releaseDownload = {tag: release.tag_name, count: 0};

          for (const asset of release.assets) {
            releaseDownload.count += asset.download_count;
          }

          download.push(releaseDownload);
        }

        return download;
      },
    },
    methods: {
      startAnalysis() {
        this.beginStatusClass = 'el-icon-loading';
        !this.repoURL && (this.repoURL = this.defaultURL);

        let url = this.repoURL;

        if (url.substr(0, 4) !== 'http') {
          url = 'https://' + url;
        }

        let info = gh(url);

        if (!info || !info.owner || !info.name) {
          this.$message.error('URL error!');
          this.data = [];
          this.beginStatusClass = '';

          return false;
        }

        let releaseURL = `https://api.github.com/repos/${info.owner}/${info.name}/releases?ran=` + Math.random();

        this.$http.get(releaseURL).then(response => {
          console.log(response);

          this.data = response.body;
          this.beginStatusClass = '';
        }).catch(response => {
          console.log('error', response);
          this.$message.error('Request Failed, Msg: ' + response.statusText);

          this.data = [];
          this.beginStatusClass = '';
        });
      },
    },
  };
</script>

<style type="text/css">
  body a {
    color: #263238;
  }
  .container{
    width: 1000px;
    margin: 10px auto;
    /*border: 1px solid grey;*/
  }
  .search-form{
    width: 100%;
  }
  #top-total {
    margin-bottom: 10px;
  }
  .release-cards{
    margin-bottom: 15px;
  }
  .release-card-tag-name {
    margin-right: 8px;
  }
  .release-card-right {
    float: right;
    font-size: 80%;
    line-height: 19px;
    color: grey;
  }
  .common-info-container{
    margin-bottom: 20px;
  }
  .copyright-declare{
    position: fixed;;
    bottom: 5px;
    left: 10px;
    font-size: 50%;
    color: grey;
  }
  .copyright-declare a {
    color: grey;
  }
</style>