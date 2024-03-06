<template>
    <div>
        <nav class="navbar navbar-light">
            <div class="container-fluid wrapper border-bottom">
                <a href="" class="btn btn-sm btn-outline-secondary">Reset</a>
                <select class="form-select form-select-sm" v-model="wordLength" @change="onWordLengthChange" style="width:54px">
                    <option value="4">4</option>
                    <option value="5" selected>5</option>
                    <option value="6">6</option>
                    <option value="7">7</option>
                </select>
            </div>
        </nav>
        <div class="wrapper">
        <div class="d-grid gap-2">
            <div v-for="(validated_line, idx) in validatedLines" :key="`vl_${idx}`">
                <validated-line
                    :validated_line="validated_line"
                >
                </validated-line>
            </div>
            <input-line
                :line="initializedInputLine"
                :validation_feedback="validationFeedback"
            >
            </input-line>
             <div class="btn-group d-flex">
                <button class="btn btn-secondary" type="button" tabindex="-1" :disabled="youWon" @click="onClearLine">Clear</button>
                <button class="btn btn-primary" type="button" :tabindex="submitTabIndex" :disabled="youWon" @click="onValidateLine">Validate</button>
             </div>
        </div>
    </div>
        <nav class="navbar fixed-bottom navbar-light bg-light">
            <div class="container-fluid">
                <div class="copyright text-end">
                    &copy; 2022 coded by 
                    <a href="https://github.com/jwhite3854" tabindex="-1" target="_blank">jwhite3854</a>.
                </div>
            </div>
        </nav>
    </div>
</template>

<script>
import InputLine from './layouts/InputLine';
import ValidatedLine from './layouts/ValidatedLine';


export default {
    name: "App",
    components: {
        "input-line": InputLine,
        "validated-line": ValidatedLine
    },
    data: function() {
        return {
            wordLength: 5,
            word: '',
            youWon: false,
            inputLine: [],
            validating: false,
            inputValidated: false,
            validatedLines: [],
            validationFeedback: null,
        }
    },
    created() {
        this.onWordLengthChange();
    },
    computed: {
        initializedInputLine() {
            return this.inputLine;
        },
        submitTabIndex() {
            return this.word.length + 1;
        }
    },
    methods: {
        onWordLengthChange() {
            this.getInitialLine();
            this.getSecretWord();
        },

        getSecretWord() {
            fetch('/wordser/random.php?len=' + this.wordLength, {
                method: 'get',
                headers: {"Accept": "application/json"}
            }).then(function(response){
                return response.json();
            }).then((results) => {
                this.word = results;
            }).catch(function (error){
                return false;
            })

        },

        getInitialLine() {
            this.inputLine = [];
            for (let ii = 0; ii < this.wordLength; ii++) {
                this.inputLine.push({value: '', error: ''});
            }
        },
         
        onClearLine() {
            let n = this.inputLine.length;
            for (let ii = 0; ii < n; ii++) {
                this.inputLine[ii].value = '';
                this.inputLine[ii].error = '';
            }
        },

        onValidateLine() {
            this.validationFeedback = null;
            let valid = this.isInputValid();
            if (!valid) {
                return;
            }
        },
        
        isInputValid() {
            this.validating = true;
            let word = '';
            for (let letter of this.inputLine) {
                if (letter.value === '') {
                    letter.error = 'is-invalid';
                } else {
                    word += letter.value;
                }
            }

            if (word.length !== this.word.length) {
                this.validationFeedback = 'Missing some letters';
                return false;
            }

            this.validateWord(word);
        },

        validateWord(word) {
            fetch('https://api.datamuse.com/words?sp='+word+'&md=d&max=1', {
                method: 'get',
            }).then(function(response){
                return response.json();
            }).then((list) => {
                if (list[0].hasOwnProperty('defs') && list[0].word === word) {
                    this.addWord();
                } else {
                    this.validationFeedback = 'Not a valid word';
                    for (let letter of this.inputLine) {
                        letter.error = 'is-invalid';
                    }
                }
            }).catch(function (error){
                return false;
            })
        },

        addWord() {
            let wordPool = this.word.toUpperCase().split('');
            let line = [];

            let n = this.inputLine.length;
            let win = 0;
            for (let ii = 0; ii < n; ii++) {
                let letter = JSON.parse(JSON.stringify(this.inputLine[ii].value.toUpperCase()));
                let position = wordPool.indexOf(letter); 
                let result = ( position > -1 ? 'warning' : 'secondary');
                if (position === ii) {
                    result = 'success';
                    win++;
                }

                line.push({letter: letter.toUpperCase(), class: "btn-"+result});
            }

            this.validatedLines.push(line);
            this.onClearLine();

            let element = document.querySelector('[tabindex="1"]');
            element.focus();

            if (win === n) {
                this.youWon = true;
                this.inputLine = [];
            }
        }
    }
}
</script>
<style>
</style>