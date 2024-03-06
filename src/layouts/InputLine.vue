<template>
    <div class="input-group has-validation">
        <input type="text" class="form-control"
            v-for="(l, idx) in line"
            :key="`l_${idx}`"
            :tabindex="idx + 1"
            v-model="l.value"
            :class="l.error"
            @change="handleLetter(l, idx)"
            @keyup="handleLetter(l, idx)"
        />
        <div class="invalid-feedback">{{ validation_feedback }}</div>
    </div>
</template>
<script>

export default {
    name: "InputLine",
    props: {
        line: {
            type: null
        },
        validation_feedback: {
            type: String
        }
    },
    methods: {
        handleLetter(input, tabIdx) {
            if (input.value.length < 1) {
                return;
            }
            input.error = '';
            let letter = input.value[0];
            input.value = letter.replace(/[^A-Za-z]/g, "");

            let element = document.querySelector('[tabindex="'+ String(parseInt(tabIdx) + 2) +'"]');
            element.focus();
        }
    }
}
</script>
<style>
</style>