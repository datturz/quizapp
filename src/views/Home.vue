<template>
  <main class="flex h-screen items-center justify-center bg-gray-100">
    <!-- quiz over -->
    <QuizComplete v-if="endOfQuiz" :percent="percentageScore"  @restartQuiz="onQuizStart"></QuizComplete>
  <!-- quizcontaines-->
  <div class="overflow-hidden bg-white flex-none container relative shadow-lg rounded-lg px-12 py-6">
    <img src="@/assets/images/cover.png" alt="" class="absolute -top-2 left-0 object-none">
    <!-- contents -->
   <div class="relative z-20">
       <!--score container-->
    <div class="text-right text-gray-800">
      <p class="text-sm leading-3">Score</p>
      <p class="font-bold">{{ score }}</p>
    </div> 
    
    <!--timercontainer-->
    <div class="bg-white shadow-lg p-1 rounded-full w-full h-5 mt-4">
      <div class="bg-blue-700 rounded-full w-11/12 h-full" :style="`width:${timer}%`"></div>
    </div>
    <!--question container-->
     <div class="rounded-lg bg-gray-100 p-2 box-question text-center font-bold text-gray-800 mt-8sssss">
          <div class="bg-white p-5">
           {{formattedQuestion}}
          </div>
     </div>

     <!--optioncontainer-->
     <div class="mt-8">
     <div v-for="(choice , item) in currentQuestion.choices" :key="item" >
       <!--optioncontainer-->
       <div class="box-question option-default bg-gray-100 p-2 rounder-lg mb-3 relative" :ref="optionChosen" @click="optionClicked(choice, item)">
          <div class="bg-blue-700 p-1 transform rotate-45 rounded-md h-10 w-10 text-white font-bold absolute right-0 top-0 shadow-md">
            <p class="transform -rotate-45">+10</p>
          </div>
          <div class="rounded-lg font-bold flex p-2 ">
            <!-- optionid-->
            <div class="p-3 rounded-lg ">{{ item }}</div>
            <!--optionid-->
            <!--optionname-->
            <div class="flex items-center pl-6">{{choice}}</div>
            <!--optionname-->
          </div>
        </div>
      <!--optioncontainer-->
     </div>
     </div>

     <!--progress indicator container-->
     <div class="mt-8 text-center">
      <div class="h-1 w-12 bg-gray-800 rounded-full mx-auto"></div>
      <p class="font-bold text-gray-8">{{questionCounter}}/{{ question.length }}</p>
     </div>
   </div>
  </div>
  </main>
</template>

<style scoped>
.box-question {
  box-shadow: 6px 6px 18px rgba(0, 0, 0, 0.09), -6px -6px 18px #ffffff; 
}
.container{
  max-width: 400px;
  border-radiu:25px;
}
</style>
<script>
import {onMounted, ref} from 'vue';
import QuizComplete from './component/QuizComplete.vue'
export default {
  setup(){
    let canClickQuestion = true
    let questionCounter = ref(0)
    let score = ref(0)
    let timer = ref(100)
    let endOfQuiz= ref(false)
    const currentQuestion = ref({
      question: "",
      answer: 1,
      choices: []
    });
    const question =ref([]);
    let percentageScore = ref(0);
    
    const loadQuestion = () => {
      canClickQuestion = true
     //check array length question 
     console.log()
     if (question.value.length > questionCounter.value ){
      timer.value = 100;
      currentQuestion.value = question.value[questionCounter.value];
      questionCounter.value++;
     } else {
      onQuizEnd()
      console.log('pertanyaan habis bang')
     }
    }
    // method
    let itemsRef = []
    const optionChosen = (element) => {
      if (element) {
        itemsRef.push(element)
      }
    }
    const clearSelected = (divSelected) => {
      setTimeout(() => {
        divSelected.classList.remove('option-correct')
        divSelected.classList.remove('option-wrong')
        divSelected.classList.add('option-default')
        loadQuestion();
      }, 1000);

    }

    const optionClicked = (choice, item) => {
      if (canClickQuestion){
        const divContainer = itemsRef[item]
      const optionID = item+1
      if (currentQuestion.value.answer == optionID){
        console.log('benarr mamang')
        score.value += 10
        divContainer.classList.add('option-correct')
        divContainer.classList.remove('option-default')
      } else {
        divContainer.classList.add('option-wrong')
        divContainer.classList.remove('option-default')
      }
      timer.value = 100;
      canClickQuestion = false
      //todoloadnextquestion
      clearSelected(divContainer)
      console.log(choice, item)
      } else {

      }
    
    }
    const countDownTimer = function (){
    let interval = setInterval(() => {
      if (timer.value > 0){
        timer.value--;
      }else {
        console.log('waktu habis bang')
        onQuizEnd()
        clearInterval(interval);
      }
       
      }, 150);

    }
    const getQuestion = async function(){
      fetch('https://opentdb.com/api.php?amount=10&category=18').then((res)=> {
        return res.json()
      })
      .then((data) => {
        const newQuestion = data.results.map((questionFromTriv) => {
          const reformatedQuestion = {
            question : questionFromTriv.question,
            choices: "",
            answer: ""
          }
          const choices = questionFromTriv.incorrect_answers;
          
          reformatedQuestion.answer = Math.floor(Math.random() * 4 + 1);
          
          choices.splice(reformatedQuestion.answer-1, 0 ,questionFromTriv.correct_answer)
          
          reformatedQuestion.choices = choices
          return reformatedQuestion
        })

        question.value = newQuestion
        loadQuestion()
        countDownTimer()
      })
    }

    const onQuizEnd = function(){
      //calculatePersentage
      percentageScore.value = (score.value / 100) * 100;
      // stop timers
      timer.value = 0;
      // show overlay
      endOfQuiz.value = true;
    }
    const onQuizStart = function() {
      //1 . set default values
      canClickQuestion = true;
      timer.value = 100;
      endOfQuiz.value = false;
      questionCounter.value = 0;
      score.value = 0;
      currentQuestion.value = {
        question: "",
        answer: 1,
        choices: [],
      };
      percentageScore.value = 0;
      question.value = [];
      //2. fetch questions from server
      getQuestion();
    };
    //lifecycle hook
    onMounted(() => {
      getQuestion()
    });


    //return
    return {
      timer,
      currentQuestion, 
      question, 
      questionCounter, 
      loadQuestion, 
      optionClicked,
      optionChosen,
      clearSelected,
      score,
      endOfQuiz,
      onQuizEnd,
      percentageScore,
      onQuizStart
    }
  },
  computed: {
    formattedQuestion() {
      let entities = {
        amp: "&",
        apos: "'",
        "#x27": "'",
        "#x2F": "/",
        "#39": "'",
        "#47": "/",
        lt: "<",
        gt: ">",
        nbsp: " ",
        quot: '"',
        "#039": "'",
      };
      return this.currentQuestion.question.replace(/&([^;]+);/gm, function(
        match,
        entity
      ) {
        return entities[entity] || match;
      });
    },
  },
  components: {
    QuizComplete,
  }
}
</script>