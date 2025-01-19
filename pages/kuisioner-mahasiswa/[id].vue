<script setup lang="ts">
import { useAuthStore } from "@/stores/auth"; // Sesuaikan dengan path store Anda
import { useCommonStore } from "@/stores/common"; // Sesuaikan dengan path store Anda
import { computed, onMounted, ref } from "vue";
import { useRoute } from "vue-router";

const { confirmDialog } = useCommonStore();
const route = useRoute();
const kusionerId = computed(() => route.params.id);

const questions = ref([
  {
    id: 1,
    kuisioner_id: 1,
    nomer: 1,
    soal: "WHY",
    jawaban_a: "A",
    jawaban_b: "B",
    jawaban_c: "C",
    jawaban_d: "D",
    jawaban_e: "E",
    jawaban_benar: "A",
    created_at: "2025-01-17T18:06:15.601Z",
    updated_at: "2025-01-17T18:06:15.601Z",
    kuisioner_title: "Pertemuan 1 Bahasa Arab Dosen A",
    kuisioner_description: "test",
    mata_kuliah_name: "Bahasa Arab",
  },
  {
    id: 2,
    kuisioner_id: 1,
    nomer: 2,
    soal: "HOW",
    jawaban_a: "A",
    jawaban_b: "B",
    jawaban_c: "C",
    jawaban_d: "D",
    jawaban_e: "E",
    jawaban_benar: "B",
    created_at: "2025-01-17T18:06:32.705Z",
    updated_at: "2025-01-17T18:06:32.705Z",
    kuisioner_title: "Pertemuan 1 Bahasa Arab Dosen A",
    kuisioner_description: "test",
    mata_kuliah_name: "Bahasa Arab",
  },
  {
    id: 3,
    kuisioner_id: 1,
    nomer: 3,
    soal: "WHAT",
    jawaban_a: "A",
    jawaban_b: "B",
    jawaban_c: "C",
    jawaban_d: "D",
    jawaban_e: "E",
    jawaban_benar: "C",
    created_at: "2025-01-17T18:08:55.398Z",
    updated_at: "2025-01-17T18:08:55.398Z",
    kuisioner_title: "Pertemuan 1 Bahasa Arab Dosen A",
    kuisioner_description: "test",
    mata_kuliah_name: "Bahasa Arab",
  },
]);

const selectedAnswers = ref({});
const currentQuestionIndex = ref(0);

onMounted(() => {
  const { user } = useAuthStore();
});

const submitAnswers = () => {
  // Logika untuk mengirim jawaban
  console.log(selectedAnswers.value);
  confirmDialog("Jawaban telah dikirim!");
};

const nextQuestion = () => {
  if (currentQuestionIndex.value < questions.value.length - 1)
    currentQuestionIndex.value++;
};

const prevQuestion = () => {
  if (currentQuestionIndex.value > 0) currentQuestionIndex.value--;
};

const progress = computed(() => {
  return ((currentQuestionIndex.value + 1) / questions.value.length) * 100;
});
</script>

<template>
  <VContainer>
    <VRow>
      <VCol cols="12">
        <VCard class="pt-3 pb-4">
          <VCardTitle>
            {{ questions[0].kuisioner_title }}
          </VCardTitle>
          <VCardSubtitle>
            Mata Kuliah: {{ questions[0].mata_kuliah_name }}
          </VCardSubtitle>
        </VCard>
      </VCol>
    </VRow>
    <VRow>
      <VCol cols="12">
        <VProgressLinear
          :model-value="progress"
          color="primary"
          height="10"
          striped
        />
      </VCol>
    </VRow>
    <VRow>
      <VCol cols="12">
        <VCard>
          <VCardText>
            <h3>
              {{ questions[currentQuestionIndex].nomer }}.
              {{ questions[currentQuestionIndex].soal }}
            </h3>
            <VRadioGroup
              v-model="selectedAnswers[questions[currentQuestionIndex].id]"
              :name="`question-${questions[currentQuestionIndex].id}`"
            >
              <VRadio
                :label="questions[currentQuestionIndex].jawaban_a"
                :value="questions[currentQuestionIndex].jawaban_a"
              />
              <VRadio
                :label="questions[currentQuestionIndex].jawaban_b"
                :value="questions[currentQuestionIndex].jawaban_b"
              />
              <VRadio
                :label="questions[currentQuestionIndex].jawaban_c"
                :value="questions[currentQuestionIndex].jawaban_c"
              />
              <VRadio
                :label="questions[currentQuestionIndex].jawaban_d"
                :value="questions[currentQuestionIndex].jawaban_d"
              />
              <VRadio
                :label="questions[currentQuestionIndex].jawaban_e"
                :value="questions[currentQuestionIndex].jawaban_e"
              />
            </VRadioGroup>
          </VCardText>
        </VCard>
      </VCol>
    </VRow>
    <VRow>
      <VCol cols="6">
        <VBtn
          color="secondary"
          :disabled="currentQuestionIndex === 0"
          @click="prevQuestion"
        >
          Sebelumnya
        </VBtn>
      </VCol>
      <VCol
        v-if="currentQuestionIndex === questions.length - 1"
        cols="6"
        class="text-right"
      >
        <VBtn color="success" @click="submitAnswers"> Kirim Jawaban </VBtn>
      </VCol>
      <VCol v-else cols="6" class="text-right">
        <VBtn
          color="primary"
          :disabled="currentQuestionIndex === questions.length - 1"
          @click="nextQuestion"
        >
          Selanjutnya
        </VBtn>
      </VCol>
    </VRow>
  </VContainer>
</template>

<style scoped>
/* Tambahkan gaya CSS jika diperlukan */
</style>
