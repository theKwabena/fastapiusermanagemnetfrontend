<template>
   
      <v-layout class="h-100">
        <v-row>
            <v-col cols="12" md="5" class="d-none d-lg-block d-md-block d-xl-block d-xxl-block">
                <div class="h-100 position-relative" style="background: linear-gradient(180deg, #3AAF9F 0%, #246B62 100%);  overflow: hidden;">
                    <div class="triangle position-absolute"></div>
                    <p class="h-text position-absolute"> Welcome </p>
                    <div class="circle position-absolute"></div>
                    <div class="triangle-mini position-absolute"></div>
                </div>
            </v-col>
            <v-col cols="12" md="7" class="d-flex justify-center align-center">
            <div class="">
                <div class="mb-10">
                    <h4 class="letter-spacing text-primary" >Login to your account</h4> 
                    <p class="text-secondary mt--2" style="font-size: 22px; font-weight: 300;">Don't have an account? <span class="font-weight-bold">
                        <router-link :to="{name : 'register'}" class="text-none text-decoration-none text-apptext">
                            Sign up
                        </router-link>
                    </span></p> 
                </div>

                <div class="mb-6 d-flex flex-column">
                    <GoogleLogin :callback="callback" prompt auto class="px-16" /> 
                        <!-- <v-btn variant="outlined" size="large" class="mb-4" prepend-icon="mdi-google">
                        Sign in with Google
                        </v-btn> -->
                    <!-- </GoogleLogin> -->
                </div>
                <div class="w-100 d-flex align-center justify-center">
                    <p class="text-center mb-n3 bg-white px-8" style="z-index: 200;">OR</p>
                </div>
                <v-divider/> 
                <p class="text-center mt-4 mb-n4 text-error" v-if="error"> {{ error }}</p>
                <div class="mb-12 mt-6">
                    <h6 class="text-subtitle-1 text-medium-emphasis">Email</h6>
                    <v-text-field 
                        density="compact"  
                        placeholder="Email address" 
                        prepend-inner-icon="mdi-email"
                        variant="outlined"
                        v-model="user.email"
                        :error-messages="v$.email.$errors.map(e => e.$message)"
                        >
                    </v-text-field>

                    <h6 class="text-subtitle-1 text-medium-emphasis d-flex align-center justify-space-between">
                        Password</h6>
                    <v-text-field
                        v-model ="user.password"
                        :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
                        :type="visible ? 'text' : 'password'"
                        density="compact"
                        placeholder="Enter your password"
                        prepend-inner-icon="mdi-lock-outline"
                        variant="outlined"
                        :error-messages="v$.password.$errors.map(e => e.$message)"
                        @click:append-inner="visible = !visible"
                    ></v-text-field>
                    <div class="d-flex justify-center align-center">
                        <v-btn 
                        :to="{'name' : 'forgot-password'}"
                        variant="text"  
                        class="text-center text-apptext text-decoration-underline  text-none text-subtitle-1"> 
                        Forgot your password?
                    </v-btn>
                    </div>
                    <div class="w-100">
                        <v-btn  elevation="0" class="bg-primary w-100" size="large" @click="login" :loading="loading">
                            Login
                        </v-btn>
                    </div>

                    
                </div>
            </div>
        
            </v-col>
        </v-row>
        

      </v-layout>
</template>

<script setup>
import {ref, reactive} from 'vue'
import { useVuelidate } from '@vuelidate/core'
import { required, email, helpers } from '@vuelidate/validators'
import { useAuthStore } from '@/store/auth.store';
import { useRouter, useRoute } from 'vue-router'
import {decodeCredential} from "vue3-google-login"


const router = useRouter()
const visible = ref(false)
const loading = ref(false)
const error = ref('')



const user = reactive({
    email : '',
    password : ''
})

const rules = {
    email : {
        required : helpers.withMessage("Please enter email", required), 
        email},
    password : {
        required : helpers.withMessage("Please enter password", required),
    }
}

// console.log(loginData)
const v$ = useVuelidate(rules, user)

const authStore = useAuthStore()


async function login(){
    error.value = ''
    loading.value = true
    const isFormCorrect = await v$.value.$validate()
    if (!isFormCorrect){
        loading.value=false
        return

    } 

    await authStore.login({ username: user.email, password:user.password })
    console.log(authStore.ready)

    if(authStore.error){
        error.value = authStore.error
        loading.value = false
    }

    if(authStore.ready){
        // localStorage.setItem('token', get_user.user.access_token)
        router.push(
            {
                name : 'home'
            }
        )
        loading.value = false
    } 
  
}


const callback = async (response) =>{
    const userData = decodeCredential(response.credential)

    await authStore.login({ username: userData.email, password:userData.sub })

    if(authStore.error){
        error.value = authStore.error
        loading.value = false
    }

    if(authStore.ready){
        // localStorage.setItem('token', get_user.user.access_token)
        router.push(
            {
                name : 'home'
            }
        )
        loading.value = false
    } 
    
}



</script>

<style scoped>
.letter-spacing {
  /* Define your desired letter spacing */
    color: #3AAF9F;
    font-size: 48px;
    font-style: black;
    font-weight: 400;
    line-height: normal;
    letter-spacing: -2.9px;
}
.log-width{
    width: 50%
}
.v-btn{
  opacity: 1
}

.triangle{
  background-color: rgba(255, 255, 255, 0.1);
  width: 500px;
  height:500px;
  transform: rotate(-101.69deg);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  left: 50%;
  bottom: 60%;
}

.circle{
  background-color: rgba(255, 255, 255, 0.1);
  width: 100px;
  height:100px;
  transform: rotate(-101.69deg);
  clip-path: circle(50% at 50% 50%);
  left: 30%;
  bottom: 20%;
}


.h-text{
    color: #FFF;
    font-size: 64px;
    font-style: normal;
    font-weight: 600;
    line-height: normal;
    top: 40%;
    left: 20%;

}

.triangle-mini{
  background-color: rgba(255, 255, 255, 0.1);
  width: 100px;
  height:100px;
  transform: rotate(-90.69deg);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  left: 85%;
  bottom: 5%;
}
</style>