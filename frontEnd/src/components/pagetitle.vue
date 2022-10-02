<template>
  <div class="pagetitle">
    <div class="logoBorder"><img src="@/assets/img/logo.png" class="logo"></div>

    <div class="title">Urban traffic|x<span style="font-weight:normal">{{ msg }}</span></div>

    <el-dropdown>
      <img class="dropDown" src="@/assets/jiao.png">
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item>Add Camera</el-dropdown-item>
        <el-dropdown-item>Add UAV</el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
    
  </div>
</template>
 
<script>
export default {
  name: 'pagetitle',
  data () {
    return {
      msg: ' emergency platform'
    }
  }
}
</script>

<style scoped>
.pagetitle{
  display: flex;
  position: fixed;
  
  top: 10px;
  left: 0;
  right: 50px;
  z-index: 100000;

  background-color:rgba(30,144,255,0);

  color: white;
}

.pagetitle img{
  display: block;
}

.logo{
  width: 80px;
  
}

.logoBorder{
  background-color:rgba(30,144,255,0);
}
.title{
  width: 30%;
  font-size: 40px;
  font-weight:lighter;
  line-height: 35px;
  /* font-family:"San Francisco", Times, serif; */
  font-family:"serif";
  margin-left: 5px;
  /* font-style:italic; */
}

.dropDown{
  width: 20px;
}


/* span{
  font-size:40px;
  font-family:"San Francisco", Times, serif;
  font-style:italic;
}
img{
  width: 40px;
  height: 40px;
}
/* .pagetitle {
  vertical-align:middle;
  height: 40px;
  width: max-content;
  position:fixed;
  background-color: black;

} */
/*#jiao {
  float: right;
}
/* .el-dropdown{
  float: right;
} */
</style>