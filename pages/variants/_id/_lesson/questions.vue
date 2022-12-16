<template>
  <div class="page">
    <div class="page-body">
      <div class="page-title font-size-24">
        <div class="back" @click="$router.push(`/variants/${variant_id}`)">
          <d-icon name="ComeBackIcon" :width="30" :height="30"/>
        </div>
        Список вопросов
        <button class="c-btn add" @click="$router.push(`/variants/${variant_id}/${lesson_id}/add`)">
          Добавить
        </button>
      </div>
      <div class="questions" v-if="questions.length>0">
        <div class="question-item" v-for="(question, i) in questions" :key="question.id">
          <div class="question-item-text">
            <span class="question-item-number">{{ i + 1 }}.</span>
            <div>
              <div v-if="question['common_question']">
                <div v-html="question['common_question'].text"></div>
                <br>
              </div>
              <div v-html="question.question"></div>
            </div>
          </div>
          <div class="question-item-answers">
            <div class="question-item-answer"
                 v-for="(answer, i) in question['answers']"
                 :key="answer.id"
                 :class="{correct: answer.correct}"
            >
              <div class="question-item-answer-letter">{{ letters[i] }})</div>
              <div class="question-item-answer-text">
                <div v-html="answer.answer"></div>
              </div>
            </div>
          </div>
          <div class="question-item-actions">
            <nuxt-link :to="{path: `/variants/${variant_id}/${lesson_id}/edit/${question.id}`}" class="edit">Изменить
            </nuxt-link>
            <button class="delete" @click="deleteQuestionModal(question.id)">Удалить</button>
          </div>
          <div class="order">
            <input type="number" v-model="question.number">
            <button @click="sendNumber(question.number, question.id)">OK</button>
          </div>
        </div>
      </div>
      <div class="questions" v-else>
        Вопросы еще не добавлены
      </div>
    </div>
  </div>
</template>

<script>
import DIcon from "~/components/core/icons/DIcon";
import {mapMutations} from "vuex";

export default {
  name: "questions",
  components: {DIcon},
  data() {
    return {
      variant_id: this.$route.params.id,
      lesson_id: this.$route.params.lesson,
      questions: [],
      isCompleted: false,
      letters: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'J', 'K'],
      isQuestionDeleteModal: false,
      questionDeleteId: null,
    }
  },
  head: {
    title: "Список вопросов",
    script: [
      {
        hid: 'stripe',
        src: 'https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML',
        defer: true,
        body: true
      }
    ]
  },
  mounted() {
    this.getStatus()
    this.getQuestions()
  },
  updated() {
    this.reRender()
  },
  methods: {
    reRender() {
      if (window.MathJax) {
        window.MathJax.Hub.Queue(["Typeset", window.MathJax.Hub], () => console.log('done'));
      }
    },
    deleteQuestionModal(id) {
      this.questionDeleteId = id
      this.isQuestionDeleteModal = true
    },
    async getQuestions() {
      this.SET_LOADER(true)
      try {
        const {data} = await this.$axios.get(`/super-admin/question-list/${this.lesson_id}/`)
        if (data) {
          this.questions = data
          await this.reRender()
        }
      } catch (e) {
        alert(e)
      } finally {
        this.SET_LOADER(false)
      }
    },
    async getStatus() {
      try {
        const data = (await this.$axios.get(`/super-admin/check-variant/${this.lesson_id}/`)).data
        this.isCompleted = data.status
      } catch (er) {
        alert(er)
      }
    },
    async sendNumber(number, id) {
      try {
        await this.$axios.patch(`/super-admin/question/${id}/`, {
          "number": number
        })
        this.$toast.success('Обновление успешно завершено!')
      } catch (er) {
        this.$toast.error('Ошибка')
      }
    },
    ...mapMutations({
      SET_LOADER: 'test/SET_LOADER'
    }),
  }
}
</script>
<style scoped lang="scss">
.c-btn {
  margin-left: auto;
  border: 1px solid #005DAE;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 500;
  font-size: 15px;
  line-height: 18px;
  color: #005DAE;
  padding: 10px 0;
  cursor: pointer;
  transition: all linear .3s;

  span {
    margin-right: 20px;
  }

  &::v-deep svg path {
    transition: all linear .3s;
  }

  &:hover {
    background-color: #005DAE;
    color: #FFFFFF;

    &::v-deep {
      svg path {
        fill: #FFFFFF !important;
      }
    }
  }

  &.add {
    width: 150px;
    background-color: #005DAE;
    color: #FFFFFF;

    &::v-deep {
      svg path {
        fill: #FFFFFF !important;
      }
    }

    &:hover {
      background-color: #FFFFFF;
      color: #005DAE;

      &::v-deep {
        svg path {
          fill: #005DAE !important;
        }
      }
    }
  }
}

.page {
  color: #000823;

  &-title {
    margin-bottom: rem(40);
    display: flex;
    align-items: center;

    .back {
      margin-right: 20px;
      cursor: pointer;

      &:hover {
        opacity: .9;
      }
    }
  }
}

.c-btn {
  margin-top: 16px;
  border: 1px solid #005DAE;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 500;
  font-size: 13px;
  line-height: 16px;
  color: #005DAE;
  padding: 10px 0;
  cursor: pointer;
  transition: all linear .3s;

  span {
    margin-right: 20px;
  }

  &::v-deep svg path {
    transition: all linear .3s;
  }

  &:hover {
    background-color: #005DAE;
    color: #FFFFFF;

    &::v-deep {
      svg path {
        fill: #FFFFFF !important;
      }
    }
  }
}

.questions {
  background: #FFFFFF;
  border: 1px solid #DEDEDE;
  box-sizing: border-box;
  border-radius: 10px;
  padding: 1.25rem;
}

.form-title {
  color: rgba(0, 0, 0, 0.5);
  margin-bottom: 1.8rem;
  font-size: 15px;
}

.form-title span {
  color: red;
  font-weight: bold;
  font-size: 18px;
}

.answer-items {

}

.item {
  display: flex;
  align-items: center;
  margin-bottom: 1.4rem;
  font-size: 17px;
}

.item .check {
  margin-right: 20px;
}

.item .letter {
  width: 40px;
}

.item input[type='text'] {
  background: #FFFFFF;
  border: 1px solid #DEDEDE;
  box-sizing: border-box;
  border-radius: 5px;
  flex-grow: 1;
  padding: .5rem;
}

.item input[type='text']::placeholder {
  color: rgba(0, 0, 0, 0.3);
  font-size: 14px;
}

.add-answer {
  color: dodgerblue;
  cursor: pointer;
  user-select: none;
}

.add-answer:hover {
  text-decoration: underline;
}

.answer-text {
  min-width: 400px !important;
  flex-grow: 1;
}

.add-form .text {
  margin-bottom: 40px;
}

.common {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
}

.common select {
  background: #FFFFFF;
  border: 1px solid #DEDEDE;
  box-sizing: border-box;
  border-radius: 3px;
  width: 200px;
  appearance: auto;
  padding: 7px 15px;
}

.common-actions {
  display: flex;
  align-items: center;
  margin-left: 20px;
}

.common-actions button {
  display: flex;
  align-items: center;
  margin-right: 15px;
  padding: 5px;
}

.common-actions button:hover {
  border-radius: 50%;
  background-color: #029aad4d;
  transition: ease .1s;
}

.form-title-common {
  color: rgba(0, 0, 0, 0.5);
  margin-bottom: .6rem;
  font-size: 15px;
}

.form-title-common span {
  color: red;
  font-weight: bold;
  font-size: 18px;
}

.add-common-text {
  margin-bottom: 10px;
}

.add-common-text input {
  background: #FFFFFF;
  border: 1px solid #DEDEDE;
  box-sizing: border-box;
  border-radius: 5px;
  padding: 7px;
  width: 100%;
}

.common-buttons {
  margin-top: 25px;
}

.common-buttons button {
  margin-right: 20px;
  border: 1px solid #029AAD;
  box-sizing: border-box;
  border-radius: 50px;
  color: #029AAD;
  padding: 5px 15px;
}

.common-buttons button:first-child {
  background-color: #029AAD;
  color: #FFFFFF;
  font-weight: bold;
}

.answer-delete {
  margin-left: 12px;
  font-size: 13px;
  padding: 5px 7px;
  background-color: #ec6e6e;
  color: #FFFFFF;
  font-weight: bold;
  border-radius: 10px;
}

.question-item {
  border-bottom: 0.5px solid #DEDEDE;
  padding-bottom: 20px;
  margin-bottom: 20px;
}

.question-item:last-child {
  padding-bottom: 0;
  margin-bottom: 0;
  border: 0;
}

.question-item-text {
  display: flex;
  align-items: center;
  font-size: 17px;
  line-height: 25px;
  color: #000000;
}

.question-item-number {
  margin-right: 8px;
}

.question-item-answers {
  margin-top: 18px;
}

.question-item-answer {
  margin-bottom: 12px;
  display: flex;
  align-items: center;
}

.question-item-answer-letter {
  margin-right: 10px;
}

.question-item-answer.correct {
  color: #00940F;
  font-weight: bold;
}

.question-item-actions {
  display: flex;
  align-items: center;
}

.question-item-actions button {
  padding: 5px 10px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: bold;
}

.question-item-actions .edit {
  padding: 5px 10px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: bold;
  background-color: #00940F;
  color: #FFFFFF;
}

.question-item-actions .delete {
  margin-left: 15px;
  background-color: #ec6e6e;
  color: #FFFFFF;
}

.modal-delete-text {
  width: 100%;
  display: flex;
  min-height: 150px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.order {
  margin-top: 10px;
}
</style>