<template>
  <div :style="`display:${loadDone ? 'inline-block' : 'none'}`">
    <div v-if="loggedIn && needRegister" class="container">
      <register @complete="evaluateState()" />
    </div>
    <div v-if="loggedIn && !needRegister" class="container">
      <myPage />
    </div>

    <div v-if="!loggedIn" class="container">
      <home />
    </div>

    <footer class="footer-white">&copy;Yusei Ito 2019</footer>
  </div>
</template>

<script>
import home from "../components/home";
import register from "../components/register";
import myPage from "../components/myPage";

import firebase from "~/plugins/firebase.js";
export default {
  components: { register, myPage, home },
  data() {
    return {
      loggedIn: false,
      needRegister: true,
      userData: null,
      loadDone: false
    };
  },
  methods: {
    evaluateState() {
      firebase.auth().onAuthStateChanged(user => {
        if (user) {
          this.loggedIn = true;
          this.userData = user;

          firebase
            .firestore()
            .collection("users")
            .doc(user.uid)
            .get()
            .then(doc => {
              if (doc.exists) {
                console.log("Already has account");
                console.log(user);
                this.needRegister = false;
                this.loadDone = true;
              } else {
                console.log("Doesn't have account yet");

                this.needRegister = true;
                this.loadDone = true;
              }
            });
        } else {
          this.loggedIn = false;
          this.userData = null;
          this.loadDone = true;
        }
      });
    }
  },
  mounted() {
    this.evaluateState();
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  background-color: #ffffff;
  border-radius: 13px;
  flex-direction: column;
}

body {
  padding-top: 30px;
  padding-bottom: 60px;
  text-align: center;
  padding-right: 10px;
  padding-left: 10px;
}

.footer-white {
  padding-top: 40px;
  color: #ffffff;
}

.loading {
  z-index: 100;
  width: 100vw;
  height: 100vh;
  background-color: #156a25;
}
</style>
