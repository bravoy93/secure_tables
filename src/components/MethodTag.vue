<template>
  <div style="padding: .25em;width: 75px;text-align: center;border-radius: 3px ; color:white"
       :class="[color]">{{method}}
  </div>
</template>
<script>
  export default {
    props: {
      method: {}
    },
    name: 'method-tag',
    computed: {
      color() {
        switch (this.method) {
          case 'GET':
            return 'success';
          case 'DELETE':
            return 'error';
          case 'PUT':
            return 'warning';
          case 'POST':
            return 'blue';
        }
        return 'black';
      }
    }
  }
</script>
<style>
  * {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none
  }
</style>