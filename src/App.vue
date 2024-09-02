<script setup>
import Loading from './components/Loading.vue'
import { AI, User } from 'aonweb'
import { ref } from 'vue'


const return_text = ref('')
const showLoading = ref(false);

const prediction = async () => {
  try {
    showLoading.value = true
    let user = new User()
    let login_user = await user.islogin()
    if (!login_user) {
        login_user = await user.login()
        if (!login_user) {
            console.log("login failed,please try again later");
            showLoading.value = false
            return
        }
    }
    const ai = new AI({
      app_key: REPLACE_APP_KEY
    })
    let data = {
      "meta-llama-3-8b-instruct@iAON-internal":{
          prompt: "with smoke, half ice and half fire and ultra realistic in detail.wolf, typography, dark fantasy, wildlife photography, vibrant, cinematic and on a black background",
          max_new_tokens:512,
          prompt_template: "{prompt}"
        }
    }
    let response = await ai.prediction(["meta-llama-3-8b-instruct@iAON-internal"],data);
    showLoading.value = false
    let responseData = null
    if (response && response.code == 200 && response.data) {
      responseData = response.data[0]
    }
    console.log('responseData.result = ',responseData.result)
    if (responseData && responseData.result) {
			return_text.value = responseData.result.join("").trim();
			console.log("return_text = ", return_text)
    }
  } catch (error) {
    console.log("prediction error = ",error)
    showLoading.value = false
  }
}

</script>

<template>
  <Loading v-if="showLoading" :showLoading="showLoading" />
	<div>
		<!-- 页面内容 -->
		<div class="">
      <button class="" @click="prediction">
        <text>生成</text>
      </button>
			<div class="uni-form-item uni-column">
        <div class="res_img">{{return_text}}</div> 
			</div>
		</div>
	</div>
</template>

<style scoped>

.container {
	padding: 0 6.4vw 18.4vw;
	margin: 0;
}

</style>
