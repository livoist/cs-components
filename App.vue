<template lang="pug">
.main
  //- ScratchCard
  .scratchCard(v-for="item in fakeScratchCards")
    .scratchs
      .drawMask(:class="{ 'isRemoveMask': !item.isMask }")
        .drawBtn(
          :class="{ 'isOpenCard': item.isOpen }"
          @click="openModal('notifys', { params: { notifyState: 'confirmScratch' } })"
        )
      ScratchCard(
        :class="{ 'removeScratch': item.isScratch }"
        :id="`card${item.id}`"
        :brushOptions="brush"
        :hideOptions="hide"
        getPercentageCleared
        @percentage-update="updatePoints"
      )
      .scartchNum {{ item.bonus }}

  //- customRadios
  CustomRadios(:radios="radios" :modelTarget="resultObject" @result="getResult")

</template>

<script>
import scratchMask from '../img/scratchMask.png'
import { ScratchCard, CustomRadios } from '../components'
import { reactive } from 'vue'

export default {
  setup() {
    // ScratchCard
    const percent = ref(0)
    const hide = reactive({
      type: 'pattern',
      src: scratchMask,
      repeat: 'repeat'
    })
    const brush = reactive({
      size: 50,
      shape: 'round'
    })
    const updatePoints = (percentage) => {
      percent.value = percentage
    }

    const fakeScratchCards = [
      {
        id: 1,
        isOpen: true,
        isMask: true,
        isScratch: false,
        bonus: '0'
      },
      {
        id: 2,
        isOpen: true,
        isMask: true,
        isScratch: false,
        bonus: '0'
      },
      {
        id: 3,
        isOpen: true,
        isMask: true,
        isScratch: false,
        bonus: '0'
      }
    ]

    // CustomRadios
    const getResult = (val) => resultObject = val
    let resultObject = reactive({
      answer1: '',
      answer2: ''
    })

    const radios = reactive([
      {
        questionTitle: '問題1',
        questionContent: [
          {
            name: 'answer1',
            id: 1
          },
          {
            name: 'answer1',
            id: 2
          },
          {
            name: 'answer1',
            id: 3
          },
          {
            name: 'answer1',
            id: 4
          }
        ]
      },
      {
        questionTitle: '問題2',
        questionContent: [
          {
            name: 'answer1',
            id: 1
          },
          {
            name: 'answer1',
            id: 2
          },
          {
            name: 'answer1',
            id: 3
          },
          {
            name: 'answer1',
            id: 4
          }
        ]
      }
    ])

    return {
      hide,
      brush,
      updatePoints,
      fakeScratchCards,
      getResult,
      resultObject,
      radios
    }
  },
  components: {
    ScratchCard,
    CustomRadios
  }
}
</script>

<style>

</style>