<template>
  <v-container>
    <h1>file upload</h1>
    <v-btn class="general" color="primary">
      <i class="ic_set ic_file"></i>
      <label for="file-selector">업로드할 파일 선택</label>
      <input
        id="file-selector"
        ref="file"
        type="file"
        @change="handleFileUpload()"
        style="display: none"
      />
    </v-btn>
    <!-- <v-btn @click="upload" color="primary">upload</v-btn> -->
    <div>
      <img
        v-for="(imageUrl, idx) in albumImageList"
        :key="idx"
        style="width: 128px; height: 128px"
        :src="imageUrl"
      />
    </div>
    <ul class="list_design">
      <li v-for="(draftInfo, idx) in draftInfos" :key="idx">
        <img style="width: 128px; height: 128px" :src="draftInfo.url" />
      </li>
    </ul>

    <v-btn v-if="draftInfos.length > 0" @click="register" color="primary">
      신청하기
    </v-btn>

    <div v-if="isRegistered">
      <li
        v-for="(album, idx) in albumNameList"
        :key="idx"
        @click="viewAlbum(album)"
      >
        {{ album }}
      </li>
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
      cloudFrontHost: "http://dqre794xq0l6.cloudfront.net/",
      bucketName: "daou-emoticon-platform-eon",
      region: "ap-northeast-2", // 리전
      identityPoolId: "ap-northeast-2:3873425b-6259-466b-992a-d420a3d1b5a7",
      s3: {
        type: Object,
      },
      albumNameList: [],
      albumImageList: [],
      draftInfos: [],
      isRegistered: false,
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
      this.upload();
    },
    viewAlbum(albumName) {
      const context = this;
      this.s3.listObjects({ Prefix: albumName }, function (err, data) {
        if (err) {
          console.log(err);
          return;
        }
        context.albumImageList = data.Contents.map((content) => {
          // const href = this.request.httpRequest.endpoint.href;
          return context.cloudFrontHost + content.Key;
          // const bucketUrl = href + context.bucketName + "/";
          // return bucketUrl + content.Key;
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
      const photoKey = "test1/" + this.file.name;
      this.s3.upload(
        {
          Key: photoKey,
          Body: this.file,
          ACL: "public-read",
        },
        (err, data) => {
          if (err) {
            console.log(err);
            return alert("업로드 중에 문제가 발생했습니다: ", err.message);
          }
          alert("업로드에 성공했습니다.");
          console.log(data);
          this.draftInfos.push({
            accessKey: data.key,
            url: this.cloudFrontHost + data.key,
            eTag: data.ETag,
          });
        }
      );
    },
    register() {
      console.log("register to server");
    },
  },
};
</script>
