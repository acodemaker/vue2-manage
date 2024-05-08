<template>
    <div>
        <el-input v-model = 'content'></el-input>
        <el-button onclick="submit()">提交</el-button>
    </div>
</template>

<script>
    import {addKnowledge} from '@/api/getData'

    export default {
        data() {
            return {
                content : ''
            }
        },
        methods: {
            submit() {
                addKnowledge(content);
            }
        }
    }

</script>

<style scoped>

</style>
