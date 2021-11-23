<template>
  <v-container>
    <h1>file upload</h1>
    <input
      id="file-selector"
      ref="file"
      type="file"
      @change="handleFileUpload()"
    />
    <v-btn @click="upload" color="primary">upload</v-btn>

    <div>
      <li
        v-for="(album, idx) in albumNameList"
        :key="idx"
        @click="viewAlbum(album)"
      >
        {{ album }}
      </li>
    </div>
    <div>
      <img
        v-for="(imageUrl, idx) in albumImageList"
        :key="idx"
        style="width: 128px; height: 128px"
        :src="imageUrl"
      />
    </div>
  </v-container>
</template>

<script>
import AWS from "aws-sdk";
export default {
  data() {
    return {
      file: {
        type: Object,
      },
      bucketName: "daou-emoticon-platform-eon",
      region: "ap-northeast-2", // 리전
      identityPoolId: "ap-northeast-2:3873425b-6259-466b-992a-d420a3d1b5a7",
      s3: {
        type: Object,
      },
      albumNameList: [],
      albumImageList: [],
    };
  },
  mounted() {
    AWS.config.update({
      region: this.region,
      credentials: new AWS.CognitoIdentityCredentials({
        IdentityPoolId: this.identityPoolId,
      }),
    });
    this.s3 = new AWS.S3({
      apiVersion: "2006-03-01",
      params: { Bucket: this.bucketName },
    });
    this.list();
  },
  methods: {
    handleFileUpload() {
      console.log("handle file");
      this.file = this.$refs.file.files[0];
    },
    viewAlbum(albumName) {
      const context = this;
      this.s3.listObjects({ Prefix: albumName }, function (err, data) {
        if (err) {
          console.log(err);
          return;
        }
        context.albumImageList = data.Contents.map((content) => {
          const href = this.request.httpRequest.endpoint.href;
          const bucketUrl = href + context.bucketName + "/";
          return bucketUrl + content.Key;
        }).filter((url) => url.charAt(url.length - 1) != "/");
        console.log(context.albumImageList);
      });
    },
    list() {
      this.s3.listObjects({ Delimiter: "/" }, (err, data) => {
        if (err) {
          console.log(err);
          return;
        }
        this.albumNameList = data.CommonPrefixes.map((album) => album.Prefix);
        console.log(this.albumNameList);
      });
    },
    upload() {
      const photoKey = "test2/" + this.file.name;
      this.s3.upload(
        {
          Key: photoKey,
          Body: this.file,
          ACL: "public-read",
        },
        (err, data) => {
          if (err) {
            console.log(err);
            return alert("there was an error: ", err.message);
          }
          alert("Success");
          console.log(data);
        }
      );
    },
  },
};
</script>
