<template>
    <div id="login">
        <transition name="fade">
            <div v-if="performingRequest" class="loading">
                <p>Loading...</p>
            </div>
        </transition>
        <section>
            <div class="col1">
                <h1>Vuegram</h1>
                <p>Welcome to the <a href="https://savvyapps.com/" target="_blank">Savvy Apps</a> sample social media web app powered by Vue.js and Firebase.
                    Build this project by checking out The Definitive Guide to Getting Started with Vue.js</p>
            </div>
            <div class="col2" :class="{ 'signup-form': !showLoginForm }">
                <form if-v="showLoginForm" @submit.prevent>
                    <h1>Welcome Back</h1>

                    <label for="email1">Email</label>
                    <input v-model.trim="loginForm.email" type="text" placeholder="you@email.com" id="email1" />

                    <label for="password1">Password</label>
                    <input v-model.trim="loginForm.password" type="password" placeholder="******" id="password1" />

                    <button @click="login" class="button">Log In</button>

                    <div class="extras">
                        <a>Forgot Password</a>
                        <a>Create an Account</a>
                    </div>
                </form>

                <form v-else @submit.prevent>
                    <h1>Get Started</h1>

                    <label for="name">Name</label>
                    <input v-model.trim="signupForm.name" type="text" placeholder="Savvy Apps" id="name" />

                    <label for="title">Title</label>
                    <input v-model.trim="signupForm.title" type="text" placeholder="Company" id="title" />

                    <label for="email2">Email</label>
                    <input v-model.trim="signupForm.email" type="text" placeholder="you@email.com" id="email2" />

                    <label for="password2">Password</label>
                    <input v-model.trim="signupForm.password" type="password" placeholder="min 6 characters" id="password2" />

                    <button @click="signup" class="button">Sign Up</button>

                    <div class="extras">
                        <a>Back to Log In</a>
                    </div>
                </form>
                <transition name="fade">
                    <div v-if="errorMsg !== ''" class="error-msg">
                        <p>{{ errorMsg }}</p>
                    </div>
                </transition>
            </div>
        </section>
    </div>
</template>

<script>
    const fb = require('../firebaseConfig.js')

    export default {
        data() {
            return {
                loginForm: {
                    email: '',
                    password: ''
                },

                signupForm: {
                    name: '',
                    title: '',
                    email: '',
                    password: ''
                },
                performingRequest : false,
                showLoginForm : true,
                errorMsg : ''
            }
        },

        methods : {

            toggleForm () {
                this.errorMsg = ''
                this.showLoginForm = !this.showLoginForm
            },

            login() {
                this.performingRequest = true
                fb.auth.signInWithEmailAndPassword(this.loginForm.email, this.loginForm.password).then(user => {
                    this.$store.commit('setCurrentUser', user)
                    this.$store.dispatch('fetchUserProfile')
                    this.$router.push('/dashboard')
                    this.performingRequest = false
                    this.errorMsg = ''
                }).catch(err => {
                    console.log(err)
                    this.performingRequest = false
                    this.errorMsg = err.message
                })
            },

            signup() {
                this.performingRequest = true
                fb.auth.createUserWithEmailAndPassword(this.signupForm.email, this.signupForm.password).then(user => {
                    this.$store.commit('setCurrentUser', user)

                    // create user obj
                    fb.usersCollection.doc(user.uid).set({
                        name: this.signupForm.name,
                        title: this.signupForm.title
                    }).then(() => {
                        this.$store.dispatch('fetchUserProfile')
                        this.$router.push('/dashboard')
                        this.performingRequest = false
                        this.errorMsg = ''
                    }).catch(err => {
                        console.log(err)
                        this.errorMsg = err.message
                        this.performingRequest = false
                    })
                }).catch(err => {
                    console.log(err)
                    this.errorMsg = err.message
                    this.performingRequest = false
                })
            }

        }
    }
</script>