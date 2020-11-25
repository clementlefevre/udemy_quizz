<template>
  <v-container>
    <v-select
      v-model="current_exam_key"
      :items="Object.keys(all_exams)"
      @change="setExam"
      item-text="state"
      item-value="abbr"
      label="Select"
      persistent-hint
      return-object
      single-line
    ></v-select>
    <br />
    <v-progress-linear v-model="current_progress"></v-progress-linear>
    <br />
    <h3>{{ current_title }}</h3>
    <br />
    <v-card elevation="2" v-if="!showResult | showUserResult">
      <v-list>
        <v-list-item-group v-model="defineCurrentAnswerIndex">
          <v-list-item
            v-for="(item, index) in current_answers"
            :key="index"
            :class="answerIsCorrect(index)"
            :selectable="showUserResult"
            :disabled="showUserResult"
          >
            <v-list-item-icon v-text="index + 1"></v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title
                v-text="_unescape(item.a_text)"
              ></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group> </v-list
      ><br /><v-spacer></v-spacer>
      <h3 v-if="showUserResult">
        {{ _unescape(this.current_question.prompt.explanation) }}
      </h3>

      <v-btn v-if="current_index > 0" elevation="2" @click="previousQuestion"
        >Previous</v-btn
      >

      <v-btn
        elevation="2"
        v-if="current_answer_index > -1 && !showUserResult"
        @click="submitQuestion"
        >Submit</v-btn
      >
      <v-btn
        v-if="current_answer_index > -1 && showUserResult"
        elevation="2"
        @click="submitQuestion"
        >Show Next result</v-btn
      >
    </v-card>
    <v-card v-if="showResult"
      >{{ finaleScore }} correct answers
      <v-progress-circular
        :rotate="360"
        :size="100"
        :width="15"
        :value="(finaleScore * 100) / 50"
        color="teal"
        >{{ (finaleScore * 100) / 50 }} %
      </v-progress-circular>
      (min 75% required)
      {{ (finaleScore * 100) / 50 }} % correct
    </v-card>
    <v-btn
      v-if="showResult && !showUserResult"
      elevation="2"
      @click="triggerShowUserResult"
      >Show Feedback</v-btn
    >
  </v-container>
</template>

<script>
import exam_1 from "../assets/quizzes/exam1.json";
import exam_2 from "../assets/quizzes/exam2.json";
import exam_3 from "../assets/quizzes/exam3.json";
import exam_4 from "../assets/quizzes/exam4.json";

export default {
  name: "UdemyQuizz",

  data: () => ({
    all_exams: {
      exam1: exam_1.results,
      exam2: exam_2.results,
      exam3: exam_3.results,
      exam4: exam_4.results,
    },
    current_exam: {},
    current_answer_index: -1,
    current_answers: [],
    current_correct_answer: -1,
    current_index: 0,
    total_score: 0,
    user_answers: {},
    showResult: false,
    finaleScore: 0,
    current_title: "",
    showUserResult: false,
    current_exam_key: "exam1",
  }),

  computed: {
    current_progress() {
      return (this.current_index * 100) / 50;
    },
    defineCurrentAnswerIndex: {
      get() {
        //this function will determine what is displayed in the input
        return this.showUserResult ? -1 : this.current_answer_index;
      },
      set(newVal) {
        //this function will run whenever the input changes
        this.current_answer_index = this.showUserResult ? -1 : newVal;
      },
    },
  },

  methods: {
    setExam: function () {
      this.current_exam = this.all_exams[this.current_exam_key];
      console.log("this.current_exam", this.current_exam);
      this.current_index = 0;
      this.user_answers = {};
      this.showResult = false;
      this.showUserResult = false;
      this.setNewQuestion();
    },
    answerIsCorrect: function (index) {
      if (this.showUserResult) {
        if (this.current_answers[index].a_index == 0) {
          if (this.user_answers[this.current_question.id].a_index == 0) {
            return "answer_and_response_OK";
          } else {
            return "answer_OK";
          }
        } else {
          if (
            this.current_answers[index].a_index ===
            this.user_answers[this.current_question.id].a_index
          ) {
            return "response_NOK";
          } else {
            return "answer_NOK";
          }
        }
      } else return "";
    },
    setTitle: function () {
      if (!this.showResult) {
        return this._unescape(this.current_question.prompt.question);
      } else {
        return "Your score :";
      }
    },
    _unescape: function (text) {
      let returnText = text;

      returnText = returnText.replaceAll("&#39;", `'`);
      returnText = returnText.replaceAll("&lt;", `<`);
      returnText = returnText.replaceAll("&gt;", `>`);
      returnText = returnText.replaceAll("&quot;", `"`);

      return returnText;
    },
    shuffleArr: function (array) {
      for (var i = array.length - 1; i > 0; i--) {
        var rand = Math.floor(Math.random() * (i + 1));
        [array[i], array[rand]] = [array[rand], array[i]];
      }
      return array;
    },
    previousQuestion() {
      this.current_index = this.current_index - 1;
      this.setNewQuestion();
    },
    submitQuestion: function () {
      this.user_answers[this.current_question.id] = this.current_answers[
        this.current_answer_index
      ];
      this.current_index = this.current_index + 1;

      this.computeResult();
      if (this.current_index > 49) {
        this.showResult = true;
        this.current_title = this.setTitle();
      } else {
        this.setNewQuestion();
      }
    },
    setNewQuestion: function () {
      this.current_question = this.current_exam[this.current_index];
      this.current_title = this.setTitle();
      this.current_answers = this.current_question.prompt.answers.map(
        (x, index) => {
          return { a_index: index, a_text: x };
        }
      );

      this.current_answers = this.shuffleArr(this.current_answers);
    },
    computeResult: function () {
      let result = Object.values(this.user_answers).reduce(function (
        sum,
        answer
      ) {
        return answer.a_index == 0 ? sum + 1 : sum;
      },
      0);
      this.finaleScore = result;
    },
    triggerShowUserResult: function () {
      this.showResult = false;
      this.current_index = 0;
      this.setNewQuestion();

      this.showUserResult = true;
    },
  },

  created: function () {
    this.current_exam = this.all_exams.exam1;
    this.setNewQuestion();
  },
};
</script>

<style scoped>
.answer_and_response_OK {
  background: lightseagreen !important;
  font-weight: bold;
  color: black !important;
}
.answer_OK {
  font-weight: bold;
  color: black !important;
  background: lightseagreen !important;
}
.response_NOK {
  background: white !important;
  color: red !important;
}
.answer_NOK {
  background: white !important;
  color: black !important;
}
</style>