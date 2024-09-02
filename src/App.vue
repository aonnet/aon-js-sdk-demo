<script setup>
import Loading from './components/Loading.vue'
import { AI, User } from 'aonweb'
import { ref } from 'vue'


const return_text = ref('')
const showLoading = ref(false);

const prediction_async = async () => {
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

  const options = {
    //Please replace with your own app_key.
    app_key: REPLACE_APP_KEY,
    is_async: true,
  }
  const aonet = new AI(options)

  let params = {}
  params['meta-llama-3-8b-instruct@iAON-internal'] = {
    "top_p": 0.95,
    "prompt": "Johnny has 8 billion parameters. His friend Tommy has 70 billion parameters. What does this mean when it comes to speed?",
    "temperature": 0.7,
    "system_prompt": "You are a helpful assistant",
    "length_penalty": 1,
    "max_new_tokens": 512,
    "stop_sequences": "<|end_of_text|>,<|eot_id|>",
    "prompt_template": "<|begin_of_text|><|start_header_id|>system<|end_header_id|>\n\n{system_prompt}<|eot_id|><|start_header_id|>user<|end_header_id|>\n\n{prompt}<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n",
    "presence_penalty": 0
  }
  let res = await aonet.prediction(['meta-llama-3-8b-instruct@iAON-internal'], { ...params })
  if (res.code === 200 && res.data) {
      let taskId = res.data.task_id
      let timing = setInterval(async () => {
          try {
              let res = await aonet.task(taskId)
              console.log(res)
              if (res.code === 200 && res.data) {
                  res = res.data
                  console.log('res 111 =',res)
                  if (res.status !== 0) {
                    showLoading.value = false
                    clearInterval(timing)
                  }
                  if (res.status == 3 || res.status == 4) {
                      let msg = (res.error && res.error) || 'Operation timed out'
                      console.log('res = ',res)
                  }

                  if (res.status == 2 || res.status == 1) {
                      console.log('create successfully, result:',res.result)
                      return_text.value = res.result.join("").trim();
			                console.log("return_text = ", return_text)
                  }

              }
              if (res.code === 201) {
                  clearInterval(timing)
                  console.log('res = ',res)
              }
          } catch (error) {
            showLoading.value = false
              console.log(error)
              if (timing) {
                  clearInterval(timing)
              }
          }
      }, 1000)
  }
  if (res.code === 201) {
    showLoading.value = false
    console.log('res = ',res)
  }
}

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
        <text>生成-同步</text>
      </button>
      <button class="" @click="prediction_async">
        <text>生成-异步</text>
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
