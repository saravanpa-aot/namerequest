<template>
  <v-btn :color="setup === 'consent' ? 'red' : ''" class="mt-auto margin-top-auto"
         id="reserve-submit-button"
          @click="handleSubmit" ref="reserve-submit-button">
    {{ text }}
  </v-btn>
</template>

<script lang="ts">
import { xproMapping } from '@/store/list-data/request-action-mapping'
import newReqModule from '@/store/new-request-module'
import { Component, Vue, Prop } from 'vue-property-decorator'

@Component({})
export default class ReserveSubmitButton extends Vue {
  @Prop(String) setup: string

  get entity_type_cd () {
    return newReqModule.entity_type_cd
  }
  get isAssumedName () {
    return newReqModule.isAssumedName
  }
  get location () {
    return newReqModule.location
  }
  get name () {
    return newReqModule.name
  }
  get request_action_cd () {
    return newReqModule.request_action_cd
  }
  get showActualInput () {
    return newReqModule.showActualInput
  }
  get text () {
    if (this.setup === 'cancel') {
      return 'Stop & Send To Examination'
    }

    // delete the next 3 lines when re-enabling auto and conditional approvals
    if (this.setup !== 'assumed') {
      return 'Send to Examination'
    }

    if (this.location !== 'BC' && this.setup !== 'assumed') {
      return 'Send for Examination'
    }
    switch (this.setup) {
      // @ts-ignore - typescript knows setup can only === 'assumed' at this point and gives error
      case 'consent':
        return 'Conditionally Reserve'
      // @ts-ignore - typescript knows setup can only === 'assumed' at this point and gives error
      case 'examine':
        return 'Send for Examination'
      case 'assumed':
        if (this.$xproMapping['ASSUMED'].includes(this.entity_type_cd)) {
          return 'Assume a name in BC'
        }
        return 'Pick a new Name'
      default:
        return 'Reserve and Continue'
    }
  }
  async sendToExamination () {
    await newReqModule.userClickedStopAnalysis()
    newReqModule.cancelAnalyzeName('NamesCapture')
  }

  handleSubmit () {
    let { setup, location, entity_type_cd, request_action_cd } = this

    if (setup === 'cancel') {
      this.sendToExamination()
      return
    }
    if (newReqModule.currentIssue?.issue_type) {
      if (['add_descriptive', 'add_distinctive'].includes(newReqModule.currentIssue.issue_type)) {
        if (!newReqModule.showActualInput) {
          this.$root.$emit('show-original-name')
        }
      }
    }

    let goToNames = () => {
      newReqModule.mutateSubmissionType('examination')
      newReqModule.mutateSubmissionTabComponent('NamesCapture')
    }
    newReqModule.mutateDisplayedComponent('SubmissionTabs')

    if (location !== 'BC' && setup !== 'assumed') {
      goToNames()
      return
    }

    /* the next 4 lines disable auto and condiotional approvals.  all setup types except
    'assumed' are short-circuited to call goToNames.  The assumed logic can remain in effect
    since they already go to examination per the existing logic.  to re-enable approvals
    delete the 4 lines that immediately follow this comment */
    if (setup !== 'assumed') {
      goToNames()
      return
    }

    switch (setup) {
      case 'assumed':
        if (this.$xproMapping['ASSUMED'].includes(entity_type_cd)) {
          newReqModule.mutateRequestActionOriginal(request_action_cd)
          newReqModule.mutateRequestAction('ASSUMED')
        }
        newReqModule.mutateAssumedNameOriginal()
        goToNames()
        return
      // @ts-ignore - typescript knows setup can only === 'assumed' at this point and gives error
      case 'examine':
        goToNames()
        return
      // @ts-ignore - typescript knows setup can only === 'assumed' at this point and gives error
      case 'consent':
        newReqModule.postNameRequests('conditional')
        newReqModule.mutateSubmissionType('consent')
        newReqModule.mutateSubmissionTabComponent('ApplicantInfo1')
        return
      default:
        newReqModule.mutateSubmissionType('normal')
        newReqModule.postNameRequests('reserved')
        newReqModule.mutateSubmissionTabComponent('ApplicantInfo1')
        return
    }
  }
}
</script>

<style scoped lang="sass">
.margin-top-auto
  margin-top: auto !important
</style>
