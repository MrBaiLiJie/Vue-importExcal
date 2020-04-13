<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <!-- <el-row>
  <el-button>默认按钮</el-button>
  <el-button type="primary">主要按钮</el-button>
  <el-button type="success">成功按钮</el-button>
    </el-row>-->
    <el-upload
      class="upload-demo"
      action
      :on-change="handleChange"
      :on-exceed="handleExceed"
      :on-remove="handleRemove"
      :before-remove="beforeRemove"
      :file-list="fileListUpload"
      :limit="limitUpload"
      accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"
      :auto-upload="false"
    >
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">只 能 上 传 xlsx / xls 文 件</div>
    </el-upload>
  </div>
</template>

<script>
export default {
  data() {
    return {
      imageUrl: "",
      msg: "hello,Element",
      limitUpload: 1,
      fileTemp: "",
      file:"",
      fileListUpload: []
    };
  },
  methods: {
    handleChange(file,fileList){
      // console.log(file)
      this.fileTemp = file.raw;
      if(this.fileTemp){
        // console.log(this.fileTemp.type)
        if(this.fileTemp.type == "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" ||
          this.fileTemp.type == "application/vnd.ms-excel"){
          this.importfxx(this.fileTemp)
        }else{
          this.$message({
            type:"warning",
            message:"附件格式错误，请删除后重新上传!"
          });
        }
      }
      
    },
    importfxx(obj) {
      console.log(obj)
      let _this = this;
      // 通过DOM取文件数据
      this.file = obj;
      var rABS = false; //是否将文件读取为二进制字符串
      var f = this.file;
      var reader = new FileReader();
      //if (!FileReader.prototype.readAsBinaryString) {
      FileReader.prototype.readAsBinaryString = function(f) {
        var binary = "";
        var rABS = false; //是否将文件读取为二进制字符串
        var pt = this;
        var wb; //读取完成的数据
        var outdata;
        var reader = new FileReader();
        reader.onload = function(e) {
          var bytes = new Uint8Array(reader.result);
          var length = bytes.byteLength;
          for (var i = 0; i < length; i++) {
            binary += String.fromCharCode(bytes[i]);
          }
          var XLSX = require("xlsx");
          if (rABS) {
            wb = XLSX.read(btoa(fixdata(binary)), {
              //手动转化
              type: "base64"
            });
          } else {
            wb = XLSX.read(binary, {
              type: "binary"
            });
          }
          outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]); //outdata就是你想要的东西
          this.da = [...outdata];
          let arr = [];
          this.da.map(v => {
            let obj = {};
            obj.code = v["设备ID"];
            obj.type = v["设备型号"];
            arr.push(obj);
          });
          return arr;
        };
        reader.readAsArrayBuffer(f);
      };

      if (rABS) {
        reader.readAsArrayBuffer(f);
      } else {
        reader.readAsBinaryString(f);
      }
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    },
    handleRemove(file, fileList) {
      // console.log(file)
      this.fileTemp = null;
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择1个文件，本次选择了 ${
          files.length
        } 个文件，共选择了 ${files.length + fileList.length} 个文件`
      );
    },
    
  }
};
</script>
<style scoped>
</style>
