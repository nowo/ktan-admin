<template>
    <IndexList title="产品分类一" :type="1" />
</template>

<script lang="ts" setup>
const IndexList = defineAsyncComponent(() => import('@/views/goods/components/ClassifyList.vue'))
</script>

<style lang="scss" scoped></style>
