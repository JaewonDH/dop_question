<template>
  <main>
    <div class="questionInfo">
      <div class="button" @click="onPrevious">
        <button :disabled="disabledPreButton">&lt;</button>
      </div>
      <div>
        <h2 translate="no">
          {{ questionNumber }}
          {{ unknown ? " ??" : "" }}
        </h2>
        <p class="question" v-html="question"></p>
        <p
          class="example"
          v-for="(example, index) in examples"
          :key="index"
          :class="example.class"
          @click="onAnswer(example)"
        >
          {{ `${examplePrefix[index]}. ${example.sentence}` }}
        </p>
      </div>
      <div class="button" @click="onNext">
        <button :disabled="disabledNextButton">&gt;</button>
      </div>
    </div>
    <div class="button mix-button">
      <button translate="no" @click="onMix()">M</button>
      <button translate="no" @click="onExampleMix()">EM</button>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, reactive } from "vue";
import { DOP_C02 } from "/src/dump/dop-c02.js";
import _ from "lodash";
const examplePrefix = ref(["A", "B", "C", "D", "E", "F", "G"]);
const orgDumps = DOP_C02;
let dumps = [];
const dump = dumps[0];
const question = ref("");
const examples = ref([]);
const unknown = ref("");

let answers = [];
const questionIndex = ref(0);
dumps = orgDumps;

const shuffle = (examples) => {
  // let array = _.shuffle(examples);
  let array = _.cloneDeep(examples);
  for (let index = array.length - 1; index > 0; index--) {
    const randomPosition = Math.floor(Math.random() * (index + 1));
    const temporary = array[index];
    array[index] = array[randomPosition];
    array[randomPosition] = temporary;
  }
  return array;
};

const getEquals = (orgList, destList) => {
  return orgList.filter((o) => {
    const result = destList.find((d) => {
      return o === d;
    });
    return result !== undefined;
  });
};

const setQuestion = (dump) => {
  question.value = dump.question;

  unknown.value = dump.unknown;

  const answersIndexList = dump.answers.map((o) => {
    return examplePrefix.value.findIndex((e) => o === e);
  });

  console.log("answersIndexList :>> ", answersIndexList);

  answers = dump.examples.filter((o) => {
    const result = answersIndexList.find((d) => {
      return o === dump.examples[d];
    });
    return result !== undefined;
  });

  // examples.value = shuffle(dump.examples);
  examples.value = dump.examples;
  examples.value = examples.value.map((e) => {
    return {
      sentence: e,
      class: "",
    };
  });
};

const onPrevious = () => {
  questionIndex.value -= 1;
  if (questionIndex.value > -1) {
    setQuestion(dumps[questionIndex.value]);
  } else {
    questionIndex.value = 0;
  }
};

const onNext = () => {
  questionIndex.value += 1;
  if (dumps.length > questionIndex.value) {
    setQuestion(dumps[questionIndex.value]);
  } else {
    questionIndex.value = dumps.length - 1;
  }
};

const disabledPreButton = computed(() => {
  return questionIndex.value === 0;
});

const disabledNextButton = computed(() => {
  return dumps.length - 1 === questionIndex.value;
});

const questionNumber = computed(() => {
  return `Question #${questionIndex.value + 1}`;
});

const onAnswer = (example) => {
  const find = answers.find((a) => {
    return a === example.sentence;
  });
  example.class = find === undefined ? "fail" : "success";
};

const onMix = () => {
  dumps = shuffle(orgDumps);
  questionIndex.value = 0;
  setQuestion(dumps[questionIndex.value]);
};

const onExampleMix = () => {
  setQuestion(dumps[questionIndex.value]);
};

setQuestion(dumps[questionIndex.value]);
</script>

<style scoped>
.questionInfo {
  display: flex;
  font-size: 20px;
}

.example {
  margin-bottom: 12px;
  cursor: pointer;
  padding: 5px;
}

.success {
  border: 1px solid blue;
}

.fail {
  border: 1px solid red;
}

.question {
  margin-bottom: 40px;
}

.button {
  display: flex;
  align-items: center;
}

button {
  font-size: 22px;
  cursor: pointer;
  width: 50px;
  height: 50px;
  margin: 50px;
}

.answer {
  padding: 10px;
  border: 1px solid black;
}
</style>
