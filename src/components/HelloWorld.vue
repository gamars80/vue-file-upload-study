<template>
  <v-container>
    <h1>파일 리스트</h1>
    <div v-for="(file,index) in fileList" :key="file.Key">
      #{{index+1}} {{file.Key}}
      <v-btn @click="deleteFile(file.Key)" color="red" flat icon>x</v-btn>
    </div>
    <h1>파일 업로드</h1>
    <input id="file-selector" ref="file" type="file" @change="handleFileUpload()">
    <v-btn @click="uploadFile" color="primary">업로드</v-btn>
  </v-container>
</template>

<script>
import AWS from "aws-sdk";

export default {
  data() {
    return {
      file: null,
      albumBucketName: "aaaa",
      bucketRegion: "222",
      IdentityPoolId: "333",
      fileList: []
    };
  },
  created() {
    this.gertFiles();
  },
  methods: {
    handleFileUpload() {
      this.file = this.$refs.file.files[0];
      console.log(this.file, "파일이 업로드 되었습니다");
    },
    uploadFile() {
      alert("dasdsa");
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      const s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: {
          Bucket: this.albumBucketName
        }
      });

      let photoKey = this.file.name;
      s3.upload(
        {
          Key: photoKey,
          Body: this.file,
          ACL: "public-read"
        },
        (err, data) => {
          if (err) {
            console.log(err);
            return alert(
              "There was an error uploading your photo:",
              err.message
            );
          }
          alert("Successfully uploaded photo");
          console.log(data);
          this.gertFiles();
        }
      );
    },
    gertFiles() {
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      const s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: {
          Bucket: this.albumBucketName
        }
      });

      s3.listObjects(
        {
          Delimiter: "/"
        },
        (err, data) => {
          if (err) {
            return alert(
              "There was an error listing your albums:" + err.message
            );
          } else {
            this.fileList = data.Contents;
            console.log(data);
          }
        }
      );
    },
    deleteFile(key) {
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      const s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: {
          Bucket: this.albumBucketName
        }
      });

      s3.deleteObject(
        {
          Key: key
        },
        (err, data) => {
          if (err) {
            return alert(
              "There was an error deleting your photo:" + err.message
            );
          }
          alert("Suceessfully deleted photo");
          this.gertFiles();
        }
      );
    }
  }
};
</script>

