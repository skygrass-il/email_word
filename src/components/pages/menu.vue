<template>
  <div>
    
    <ul class="nav">
  <li class="nav-item">
    <router-link class="nav-link" to="/page/">卡片1</router-link>
  </li>
  <li class="nav-item">
    <router-link class="nav-link" to="/page/child2">卡片2</router-link>
  </li>
  <li class="nav-item">
     <router-link class="nav-link" to="/page/child3">卡片3</router-link>
  </li>
    <li class="nav-item">
     <a href="#" class="nav-link" @click.prevent="updatePath">切換指定頁面</a>
  </li>
    <li class="nav-item">
     <a href="#" class="nav-link" @click.prevent="beforePath">回到前頁</a>
  </li>
</ul>

  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {

    }
  },
  methods:{
      updatePath(){
          this.$router.replace('/page/child2');
      },
      beforePath(){
          this.$router.go(-1);
      },
  }
}
</script>


