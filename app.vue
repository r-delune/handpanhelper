<template>
  <div>
    <!-- <NuxtWelcome /> -->

    dropdown - select key



    {{ first }}
    {first}
  </div>
</template>
<script>
import { ref, computed } from "vue"
export default {
  name: 'IndexPage',
    setup() {

    return {
      first: ref(1),

    }
  },
}
</script>