<script setup lang="ts">
import supabaseClient from '@/utils/superbase'
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'

import PageWrapper from '@/components/PageWrapper.vue'

const quizSession = ref<any>(null)
const loading = ref(true)
const isFinished = ref(false)
const { push } = useRouter()
onMounted(() => {
  const getSession = async () => {
    try {
      loading.value = true
      const {
        data: { user }
      } = await supabaseClient.auth.getUser()
      if (!user) return
      const existingSession = await supabaseClient
        .from('quiz_session')
        .select('id, last_attempted, is_finished')
        .eq('user_id', user.id)
      if (existingSession.data) {
        quizSession.value = existingSession.data[0]
        isFinished.value = !!existingSession.data[0].is_finished
      }
    } catch (error) {
      console.log(error)
    } finally {
      loading.value = false
    }
  }
  getSession()
})
const startQuiz = async () => {
  try {
    loading.value = true
    const user = await supabaseClient.auth.getUser()
    console.log(user)
    if (!user.data?.user) return
    const quizData = await supabaseClient
      .from('quiz_session')
      .insert({
        user_id: user.data.user.id,
        is_started: true
      })
      .select()
    push(`/quiz/${quizData.data?.[0].id}/ongoing/1`)
  } catch (error) {
    console.log(error)
    loading.value = false
  }
}
const continueQuiz = () => {
  if (!quizSession.value) return
  push(`/quiz/${quizSession.value.id}/ongoing/${quizSession.value.last_attempted}`)
}
const onClick = () => {
  if (quizSession.value) {
    continueQuiz()
  } else {
    startQuiz()
  }
}
</script>

<template>
  <PageWrapper>
    
    <main class="flex-1 px-4 flex justify-center items-center">
      <div
        v-if="loading"
        class="border-2 border-black animate-spin border-r-transparent w-10 h-10 rounded-full"
      ></div>
      <button
        class="bg-black rounded-lg py-4 text-center font-bold text-lg text-white w-full max-w-sm"
        @click="onClick"
        v-if="!loading && !isFinished"
      >
        {{ quizSession ? 'Continue Quiz' : 'Start quiz' }}
      </button>
      <button
        class="bg-black rounded-lg py-4 text-center font-bold text-lg text-white w-full max-w-sm"
        v-if="!loading && quizSession && isFinished"
        @click="
          () => {
            // router.addRoute({ path: `/quiz/result?quizId=${quizSession.id}`, component: SingleQuiz })
            push(`/result/${quizSession.id}`)
          }
        "
      >
        View Result
      </button>
    </main>
  </PageWrapper>
</template>
