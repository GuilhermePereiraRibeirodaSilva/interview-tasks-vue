<script setup lang="ts">
/*
* OVERALL COMMENTS:
* There were several problems regarding the names used for funtions and variables in the code which were very confusing,
* the reactivity which was not being properly used since direct DOM manipulation was being used, the lack of true randomness
* in the algoritmh among others.
* 
* With the current changes the it should be much more readable and understandable. It's all commented so the code choices
* are explained. If you have any questions please contact me via my email: guinovembro43@gmail.com
*/



// !SECTION IMPORTS
import { ref,computed,watch,onMounted } from 'vue';

// !SECTION VARS
//Let's make numbers a reactive variable, so we can use it in the template
const numbers = ref([]);

//Creating a a property for the errors
const errors = ref([])

//This must be a reactive variable, so we can use it in the template
let limit = ref(100);

// !SECTION FUNCTIONS
//This function uses the Fisher-Yates algorithm, much more random.
function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

//Again, this function is not descriptive at all, a better name would be highlightDivisors
function highlightDivisors(number) {
  //Very bad practice to manipulate the DOM directly, we should use Vue's reactivity system instead
  numbers.value = numbers.value.map(num => ({
	number: num.number,
	active: number % num.number === 0
  }));
}

function reset()
{
	//Again, very bad practice to manipulate the DOM directly, we should use Vue's reactivity system instead
	numbers.value = numbers.value.map(num => ({
		number: num.number,
		active: false
	}));
}

function fillNumbers(){
	numbers.value = shuffle([...Array(limit.value).keys()]
	.map(i => ({
		number: i + 1, //1 - 100 not 0 - 99
		active: false
	})));
}

// !SECTION LIFECYCLE
onMounted(() => {
	fillNumbers()
});


// !SECTION WATCHERS
//n() is a bad name, not descriptive at all, a better name would be shuffledNumbers
//Lets watch the limit varibale so we can update the numbers when it changes
watch(limit, (newLimit) => {
	errors.value = [];
	//Making sure the limit is between 1 and 100
	if (newLimit < 1 || newLimit > 100) {
		errors.value.push('Limit must be between 1 and 100');
		numbers.value = [];
		return;
	}

	fillNumbers()
});
</script>

<template>
	<div>
		<input type="number" v-model="limit" />
		<div class="errors">
			<div v-for="(error, index) in errors" :key="index" class="error">{{ error }}</div>
		</div>
		<div 
			:class="['number', numberInfo.active ? 'active' : '']"
			v-for="(numberInfo, index) in numbers"
			:key="index"
			@mouseover="highlightDivisors(numberInfo.number)"
			@mouseout="reset"
		>
			{{ numberInfo.number }}
		</div>
	</div>
</template>

<style scoped>
.number {
	display: inline-block;
	padding: 5px;
	background-color: lightgrey;
	margin: 5px;
}

.active {
	background-color: red;
}

.errors{
	margin: 10px;
}

.error{
	color: red;
	font-weight: bold;
}
</style>
