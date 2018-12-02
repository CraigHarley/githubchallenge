<template>
  <div id="app" class="container-fluid center-block">
    <br>

    <div class="row">
      <img src="https://octodex.github.com/images/octobiwan.jpg" height="300px" class="center-block">
    </div>  

    <br>

    <div class="row">
      <div class="centered">
        <div id="search_container">
          <div class="input-group search-input-group">
            <input type="text" class="form-control"  placeholder="Search" v-on:keyup.enter="searchKeyup()" v-model="search">
            <span class="input-group-addon">
              <button type="button" v-on:click="searchButtonPressed()">
                  <span class="glyphicon glyphicon-search"></span>
              </button>
            </span>
          </div>
        </div>
      </div>
    </div>

    <br>

    <div class="row" v-if="currentView === 'results'">
      <div v-if="results.length === 0">
        <h3 class="text-center">No results yet, try searching.</h3>
      </div>
      <table class="table table-striped table-hover" v-if="results.length > 0">
        <thead>
        <tr>
          <th></th>
          <th><i class="fa fa-user" aria-hidden="true"></i></th>
          <th><i class="fa fa-eye" aria-hidden="true"></i></th>
          <th><i class="fa fa-code-fork" aria-hidden="true"></i></th>
          <th></th>
        </tr>
        </thead>
        <tbody>
          <tr v-for="repo in results" :key="repo.id">
            <td>{{repo.name}}</td>
            <td>{{repo.owner.login}}</td>
            <td>{{repo.watchers_count}}</td>
            <td>{{repo.forks}}</td>
            <td><button class="btn btn-success btn-block" type="button" v-on:click="showIndividualRepo(repo)">View</button></td>
          </tr>
        </tbody>
      </table>
    </div>


    <div class="row" v-else-if="currentView === 'single'">
      <div class="well">
        <button style="float:right" class="btn btn-info" v-on:click="goBackToResults()"><i class="fa fa-level-up" aria-hidden="true"></i>
          Go Back</button>
        <h3>More information about <strong>{{selectedRepo.name}}</strong></h3>

        <h4>
          <i class="glyphicon glyphicon-user"></i> <strong><a :href="selectedRepo.owner.url">{{selectedRepo.full_name}}</a></strong>
        </h4>
        <p>
          <span class="fixed-width"><i class="fa fa-calendar" aria-hidden="true"></i> Created:</span> {{selectedRepo.created_at}}<br>
          <span class="fixed-width"><i class="fa fa-calendar-o" aria-hidden="true"></i> Updated:</span> {{selectedRepo.updated_at}}<br>
          <span class="fixed-width"><i class="fa fa-area-chart" aria-hidden="true"></i> Size:</span> {{selectedRepo.size}}<br>
          <span class="fixed-width"><i class="fa fa-external-link" aria-hidden="true"></i> Clone Url:</span> git clone {{selectedRepo.clone_url}}<br>
          <span class="fixed-width"><i class="fa fa-code-fork" aria-hidden="true"></i> Default Branch:</span> {{selectedRepo.default_branch}}<br>
          <span class="fixed-width"><i class="fa fa-eye" aria-hidden="true"></i> Watchers:</span> {{selectedRepo.watchers}}<br>
          <span class="fixed-width"><i class="fa fa-star" aria-hidden="true"></i>Stargazers:</span> {{selectedRepo.stargazers_count}}<br>
          <span class="fixed-width"><i class="fa fa-code-fork" aria-hidden="true"></i> Forks:</span> {{selectedRepo.forks}}<br>
          <span class="fixed-width"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i> Open Issues:</span> {{selectedRepo.open_issues_count}}<br>
          <span class="fixed-width"><i class="fa fa-thumbs-up" aria-hidden="true"></i>Score:</span> <strong>{{selectedRepo.score}}</strong>
        </p>
        <p v-html="selectedRepo.readme" class="well"></p>
      </div>
    </div>
  </div>
</template>

<script>
import * as request from 'superagent';
import * as moment from 'moment';
import * as showdown from 'showdown';
const converter = new showdown.Converter();
converter.setFlavor('github');

export default {
  name: 'app',
  data () {
    return {
      search: '',
      results: [],
      selectedRepo: {},
      viewedRepos: {},
      currentView: 'results'
    }
  },
    methods : {
        showIndividualRepo: function(repo){
            if (!this.$data.viewedRepos[repo.id]){
                //lazily convert dates to human format:
                repo.created_at = moment(repo.created_at).format('MMMM Do YYYY, h:mm:ss a');
                repo.updated_at = moment(repo.updated_at).format('MMMM Do YYYY, h:mm:ss a');
                this.$set(this.$data.viewedRepos, repo.id, true);
            }

            this.$data.selectedRepo = repo;
            this.getReadmeForCurrent();
            this.$data.currentView = 'single';
        },
        getReadmeForCurrent: function(){
            const repo = this.$data.selectedRepo;
            request
                .get(`https://api.github.com/repos/${repo.owner.login}/${repo.name}/readme`)
                .then(this.getReadMeSuccess, this.handleError);
        },
        getReadMeSuccess: function(res) {
            this.$data.selectedRepo.readme = converter.makeHtml( atob( res.body.content ) );
        },
        goBackToResults: function(){
            this.$data.currentView = 'results';
        },
        searchKeyup: function(){
            this.doGithubSearch(this.$data.search);
            this.goBackToResults();
        },
        doGithubSearch: function(str){
            request
                .get(`https://api.github.com/search/repositories?q=${str}`)
                .then(this.searchSuccess, this.handleError);
        },
        searchSuccess: function(res){
            this.$data.results = res.body.items.map(x => {
                x.readme = ''; //required so vue getters/setters are attached for reactivity
                return x;
            });
        },
        handleError: function(err){
            console.log(err);
        }
    }
}
</script>

<style>
#app {
    margin: 20px;
}

 table {
     table-style: fixed;
}
 .fixed-width {
     display: inline-block;
     min-width:200px;
     max-width:200px;
}

 .search-input-group .input-group-addon {
     background: white !important;
}
 .search-input-group .form-control{
     border-right:0;
     box-shadow:0 0 0;
     border-color:#ccc;
     height: 50px;
}
 .search-input-group button {
     border:0;
     background:transparent;
}
 </style>
