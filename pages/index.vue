<template>
  <div class="container">
    <div>
      <h1 class="title">
        Github deployment cleaner
      </h1>
      <p>
        This tool performs all calls client side, no keys will be saved.
        All calls are performed directly on the Github API.
        See source code here: <a href="https://github.com/jaap/github-deployment-cleaner">https://github.com/jaap/github-deployment-cleaner</a>
      </p>
      <p>
        Based on <a href="https://stackblitz.com/edit/github-deployment-clearer">https://stackblitz.com/edit/github-deployment-clearer</a>
        And <a href="https://stackoverflow.com/questions/53452910/how-to-remove-a-github-environment">https://stackoverflow.com/questions/53452910/how-to-remove-a-github-environment</a>
      </p>
      <p>
        <strong>RECOMMENDED</strong>: Disconnect HEROKU from Github before doing this (though not strictly necessary, I think).
        <br>
        See <a href="https://stackoverflow.com/a/61272173/6569950">https://stackoverflow.com/a/61272173/6569950</a> for more info.
      </p>

      <b-form-group
        id="user_or_rog"
        label="User or organisation name"
        label-for="user_or_rog"
      >
        <b-form-input id="input-1" v-model="userOrOrg" trim></b-form-input>
      </b-form-group>

      <b-form-group
        id="repo"
        label="Repository"
        label-for="repo"
      >
        <b-form-input id="input-1" v-model="repo" trim></b-form-input>
      </b-form-group>

      <b-form-group
        id="token"
        label="Token"
        description="Must be `repo_deployments` authorized"
        label-for="token"
      >
        <b-form-input id="input-1" v-model="token" trim></b-form-input>
      </b-form-group>

      <b-form-group
      >
        <b-button @click="getDeployments">Get deployments</b-button> - Gets only 30 at a time. Repeat if needed. I'll accept a PR to do some smarter automation.
      </b-form-group>

      <table class="table" v-if="deployments.length > 0">
        <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Description</th>
          <th scope="col">Creator</th>
          <th scope="col">Created at</th>
          <th>
            <b-button variant="danger" @click="deleteAll">Delete all</b-button>
          </th>
        </tr>
        </thead>
        <tbody name="fade" is="transition-group">
          <tr v-for="deployment in deployments" :key="deployment.id" v-if="!deployment.deleted">
            <th scope="row">{{ deployment.id }}</th>
            <td>{{ deployment.description }}</td>
            <td>{{ deployment.creator.login }}</td>
            <td>{{ deployment.created_at }}</td>
            <td>
              <b-button v-if="!deployment.deleting" variant="danger" @click="deleteDeployment(deployment)">Delete</b-button>
              <b-spinner variant="danger" v-else></b-spinner>
            </td>
          </tr>
        </tbody>
      </table>

    </div>
  </div>
</template>

<script>
// GLOBAL VARS
// const URL = `https://api.github.com/repos/${USER_OR_ORG}/${REPO}/deployments`;
// const AUTH_HEADER = `token ${TOKEN}`;

export default {
  data() {
    return {
      userOrOrg:  process.env.gh_org_or_user || '',
      repo: process.env.gh_repo || '',
      token: process.env.gh_token || '',
      deployments: []
    }
  },
  computed: {
    url() {
      return `https://api.github.com/repos/${this.userOrOrg}/${this.repo}/deployments`
    }
  },
  methods: {
    async getDeployments(){
      const res = await fetch(this.url, { headers: { authorization: `token ${this.token}` } });
      const data = await res.json();
      data.forEach(function (element) {
        element.deleted = false
        element.deleting = false
      });
      this.deployments = data;
    },
    async deleteDeployment(deployment){
      deployment.deleting = true
      fetch(`${this.url}/${deployment.id}/statuses`, {
        method: "POST",
        body: JSON.stringify({ state: "inactive" }),
        headers: {
          "Content-Type": "application/json",
          Accept: "application/vnd.github.ant-man-preview+json",
          authorization: `token ${this.token}`
        }
      }).then(() => {
        fetch(`${this.url}/${deployment.id}`, {
          method: "DELETE",
          headers: { authorization: `token ${this.token}` }
        }).then(() => {
          deployment.deleting = false;
          deployment.deleted = true;
        })
      });
    },
    async deleteAll(){
      this.deployments.forEach(deployment => {
        this.deleteDeployment(deployment)
      })
    }
  }
}
</script>

<style>
.fade-item {
}
.fade-leave-active {
  transition: all 300ms;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  transform: scale(1, 0);
  height: 0;
  margin: 0;
  padding: 0;
  opacity: 0;
}
</style>
