<template>
  <div class="upload">
    <div style="margin-top: 20px;margin-left: 20px;">
      测试搜索框
      <a-select
        mode="multiple"
        labelInValue
        :value="value"
        placeholder="使用中文输入法输入英文显示不全"
        style="width:200px"
        :filterOption="false"
        @search="fetchUser"
        @change="handleChange"
        :notFoundContent="fetching ? undefined : null"
        @input="inputKeydownData"
      >
        <a-spin v-if="fetching" slot="notFoundContent" size="small" />
        <a-select-option v-for="d in data" :key="d.value"
          >{{ d.text }}
        </a-select-option>
      </a-select>
    </div>

    <input type="file" @change="JimpFile($event)">
    <img src="./logo.png" alt="" />
  </div>
</template>
<script>
// import jsonp from 'fetch-jsonp';
import Jimp from "jimp";

export default {
  data() {
    this.lastFetchId = 0;
    // this.fetchUser = debounce(this.fetchUser, 800);
    return {
      data: [],
      value: [],
      fetching: false
    };
  },
  methods: {
    JimpFile(event){
      let that=this;
     let oFile = event.target.files[0];
      var fileReader = new FileReader();
      fileReader.readAsDataURL(oFile);

      fileReader.onload = function (e) {
        var data = e.target.result;
        //加载图片获取图片真实宽度和高度
        var image = new Image();

        image.onload = function () {
          let srcBase64 = image.src
          let bufferData = that.base64ToUint8Array(srcBase64.split(",")[1]);

          console.log(bufferData)
          // Jimp.read(require('./logo.png'))
          Jimp.read(bufferData)
                  .then(image => {
                    console.log(image);
                  })
                  .catch(err => {
                    console.log(err);

                  });


        };
        image.src = data;
      };
    },
    base64ToUint8Array(base64String) {
      const padding = "=".repeat((4 - (base64String.length % 4)) % 4);
        const base64 = (base64String + padding)
        // eslint-disable-next-line no-useless-escape
            .replace(/\-/g, '+')
            .replace(/_/g, '/');

      const rawData = window.atob(base64);
      const outputArray = new Uint8Array(rawData.length);

      for (let i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
      }
      return outputArray;
    },
    inputKeydownData(val) {
      console.log(val);
    },
    fetchUser(value) {
      console.log("fetching user", value);
      this.lastFetchId += 1;
      const fetchId = this.lastFetchId;
      this.data = [];
      this.fetching = true;
      fetch("https://randomuser.me/api/?results=5")
        .then(response => response.json())
        .then(body => {
          if (fetchId !== this.lastFetchId) {
            // for fetch callback order
            return;
          }
          const data = body.results.map(user => ({
            text: `${user.name.first} ${user.name.last}`,
            value: user.login.username
          }));
          this.data = data;
          this.fetching = false;
        });
    },
    handleChange(value) {
      console.log("变化", value);
      Object.assign(this, {
        value,
        data: [],
        fetching: false
      });
    }
  },
  mounted() {
    // Jimp.read('https://pic1.zhimg.com/v2-547d4bed546613203aad6bf009a7f2e8_540x450.jpeg')
    Jimp.read(require("./logo.png"))
      .then(image => {
        console.log(image);
      })
      .catch(err => {
        console.log(err);
      });

    console.log("测试搜索框");
  }
};
</script>
