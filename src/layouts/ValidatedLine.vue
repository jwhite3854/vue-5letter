<template>
    <div class="btn-group d-flex">
        <button type="button" class="btn"
            v-for="(button, idx) in validated_line"
            :key="`b_${idx}`"
            :class="button.class"
        >{{ button.letter }}
        </button>
    </div>
</template>
<script>

export default {
    name: "ValidatedLine",
    props: {
        validated_line: {
            type: null
        }
    }
}
</script>
<style>
</style>