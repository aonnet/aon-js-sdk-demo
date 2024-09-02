# AONWEB

aonweb - JavaScript sdk for AON

- API version: 1.0.0
- Package version: 2.0.2

## Installation

### For [Node.js](https://nodejs.org/)

#### npm

Install it from npm:

```shell
npm install aonweb --save or npm install aonweb@2.0.2 --save
```

## Getting Started

Please follow the [installation](#installation) instruction and execute the following JS code:

```javascript

import { AI, User } from 'aonweb'

const prediction = async () => {
  try {
    let user = new User()
    let login_user = await user.islogin()
    if (!login_user) {
        login_user = await user.login()
        if (!login_user) {
            console.log("login failed,please try again later");
            return
        }
    }
    const ai = new AI({
      app_key: REPLACE_APP_KEY //replace app key
    })
    let data = {
      "meta-llama-3-8b-instruct@iAON-internal":{
          prompt: "with smoke, half ice and half fire and ultra realistic in detail.wolf, typography, dark fantasy, wildlife photography, vibrant, cinematic and on a black background",
          max_new_tokens:512,
          prompt_template: "{prompt}"
        }
    }
    let response = await ai.prediction(["meta-llama-3-8b-instruct@iAON-internal"],data);
    let responseData = null
    if (response && response.code == 200 && response.data) {
      responseData = response.data[0]
    }
    console.log('responseData.result = ',responseData.result)
  } catch (error) {
    console.log("prediction error = ",error)
  }
}

prediction();

```

### Constructor

name | type | Description
------------ | ------------- | -------------
*app_key* | string | **Required**.