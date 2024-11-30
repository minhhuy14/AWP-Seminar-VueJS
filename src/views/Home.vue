<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import { useCounterStore } from "../stores/counter";
import { shuffle } from "../lib/utils";

import StarButton from '../components/StartButton.vue'

const store = useCounterStore()
const router = useRouter()

const loading = ref(false)
const error = ref(false)

const category = ref('any');
const difficulty = ref('any');
const type = ref('any');
const amount = ref(10);

const processQuestions = (data) => {
    if (!data || !data.results) return null;

    return data.results.map((item, index) => {
        const answers = [item.correct_answer, ...item.incorrect_answers];
        const choices = shuffle(
            answers.map((ans, i) => ({ id: i, text: ans }))
        );
        return {
            ...item,
            id: index,
            text: item.question,
            choices,
            answer: 0,
        };
    });
};
const fetchQuestions = async () => {
    loading.value = true;
    error.value = false;

    let url = `https://opentdb.com/api.php?amount=${amount.value}`;
    if (category.value !== 'any') url += `&category=${category.value}`;
    if (difficulty.value !== 'any') url += `&difficulty=${difficulty.value}`;
    if (type.value !== 'any') url += `&type=${type.value}`;

    try {
        const response = await fetch(url);
        const data = await response.json();
        return processQuestions(data);
    } catch (err) {
        console.error(err);
        error.value = true;
        return null;
    } finally {
        loading.value = false;
    }
};


const startQuiz = async () => {
    const questions = await fetchQuestions();
    if (questions) {
        store.setQuizData(questions);
        store.resetScore();
        store.increment();
        store.setQuestionIndex(0);
        router.push('/question/0');
    }
};

onMounted(async () => {
    store.resetQuiz();
    const questions = await fetchQuestions();
    if (questions) {
        store.setQuizData(questions);
    }
});
</script>

<template>
    <div class="container">
        <form @submit.prevent="fetchQuestions">
            <div>
                <label for="amount">Number of Questions: </label>
                <input type="number" id="amount" v-model="amount" min="1" max="50" />
            </div>
            <div>
                <label for="category">Category: </label>
                <select id="category" v-model="category">
                    <option value="any">Any Category</option>
                    <option value="9">General Knowledge</option>
                    <option value="10">Entertainment: Books</option>
                    <option value="11">Entertainment: Film</option>
                    <option value="12">Entertainment: Music</option>
                    <option value="13">Entertainment: Musicals &amp; Theatres</option>
                    <option value="14">Entertainment: Television</option>
                    <option value="15">Entertainment: Video Games</option>
                    <option value="16">Entertainment: Board Games</option>
                    <option value="17">Science &amp; Nature</option>
                    <option value="18">Science: Computers</option>
                    <option value="19">Science: Mathematics</option>
                    <option value="20">Mythology</option>
                    <option value="21">Sports</option>
                    <option value="22">Geography</option>
                    <option value="23">History</option>
                    <option value="24">Politics</option>
                    <option value="25">Art</option>
                    <option value="26">Celebrities</option>
                    <option value="27">Animals</option>
                    <option value="28">Vehicles</option>
                    <option value="29">Entertainment: Comics</option>
                    <option value="30">Science: Gadgets</option>
                    <option value="31">Entertainment: Japanese Anime &amp; Manga</option>
                    <option value="32">Entertainment: Cartoon &amp; Animations</option>
                </select>
            </div>
            <div>
                <label for="difficulty">Difficulty: </label>
                <select id="difficulty" v-model="difficulty">
                    <option value="any">Any Difficulty</option>
                    <option value="easy">Easy</option>
                    <option value="medium">Medium</option>
                    <option value="hard">Hard</option>
                </select>
            </div>
            <div>
                <label for="type">Type: </label>
                <select id="type" v-model="type">
                    <option value="any">Any Type</option>
                    <option value="multiple">Multiple Choice</option>
                    <option value="boolean">True / False</option>
                </select>
            </div>
        </form>
        <div v-if="!loading && !error">
            <StarButton @click="startQuiz">Start Quiz</StarButton>
        </div>
        <p v-if="loading">Fetching trivia questions....</p>
        <p v-if="error">Oops, something went wrong!</p>
    </div>
</template>

<style scoped>
.container {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}
.button {
    display: flex;
    justify-content: space-between;
    align-items: center;
}
form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

label {
    font-weight: bold;
}

input, select {
    padding: 0.5rem;
    font-size: 1rem;
}
</style>