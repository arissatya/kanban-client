<template>
  <div>
    <div class="container mx-auto align-items-center" v-if="alertOn">
      <div class="alert mt-2 ml-5 alert-warning position-absolute" role="alert">{{alertMsg}}</div>
    </div>
    <Dashboard
      v-if="isLogin"
      :allOrganizationTask="organizationTasks"
      :UserName="name"
      :UserOrganization="organization"
      @userLoggedOut="logoutClear"
      @failAddTask="registrationStatus"
      @deleteTaskFailed="registrationStatus"
      @deleteTasksucceeded="newTaskOnList"
      @refreshTasks="newTaskOnList"
      @newTaskStatus="newTaskOnList"
      @updateFailed="registrationStatus"
    ></Dashboard>
    <FrontPage
      v-else
      @loginSucceeded="loginSuccess"
      @googleLoginSucceeded="loginSuccess"
      @loginFailed="registrationStatus"
      @registrationSucceeded="registrationStatus"
      @registrationFailed="registrationStatus"
    ></FrontPage>
  </div>
</template>

<script>
import KanbanAPI from "./config/KanbanAPI";

import FrontPage from "./components/FrontPage";
import Dashboard from "./components/Dashboard";

export default {
  name: "App",
  data() {
    return {
      name: "",
      organization: "",
      alertMsg: "",
      isLogin: false,
      alertOn: false,
      organizationTasks: [],
    };
  },
  methods: {
    fetchOrganizationTask() {
      console.log("KE FETCH");
      let count = 0;
      KanbanAPI({
        method: "GET",
        url: "tasks",
        headers: {
          access_token: localStorage.getItem("access_token"),
        },
      })
        .then((response) => {
          this.organizationTasks = response.data;
        })
        .catch((err) => {
          console.log(err);
        })
        .finally((_) => {
          console.log("FETCHED TASKS");
        });
    },
    loginSuccess(data) {
      localStorage.setItem("organization", data.organization);
      localStorage.setItem("name", data.name);
      this.alertMsg = data.alertMsg;
      this.isLogin = true;
      this.alertOn = true;
      setTimeout((_) => {
        this.fetchOrganizationTask();
        this.alertOn = false;
        this.alertMsg = "";
      }, 3000);
    },
    newTaskOnList(data) {
      this.fetchOrganizationTask();
      this.alertOn = true;
      this.alertMsg = data.alertMsg;
      setTimeout((_) => {
        this.alertOn = false;
        this.alertMsg = "";
      }, 5000);
    },
    registrationStatus(data) {
      this.alertOn = true;
      this.alertMsg = data.alertMsg;
      setTimeout((_) => {
        this.alertOn = false;
        this.alertMsg = "";
      }, 3000);
    },
    logoutClear() {
      this.alertOn = true;
      this.alertMsg = "LOGGED OUT!";
      setTimeout((_) => {
        this.alertOn = false;
        this.alertMsg = "";
      }, 3000);
      this.name = "";
      this.organization = "";
      this.isLogin = false;
    },
  },
  components: {
    FrontPage,
    Dashboard,
  },
  created() {
    if (localStorage.getItem("access_token")) {
      this.fetchOrganizationTask();
      this.isLogin = true;
    } else {
      console.log("YOU ARE NOT LOGIN");
    }
  },
};
</script>

<style>
</style>