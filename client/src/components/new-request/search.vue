<template>
  <v-container fluid id="new-request-container" class="normal-copy">
    <v-row class="pa-6" justify="end">
      <v-col cols="6" class="mb-n3 normal-copy">I need a name to:</v-col>
      <v-col cols="6" style="display: flex; justify-content: flex-end" class="mb-n3">
        <button class="normal-link"
              id="help-me-choose-activator"
              @change="activateHMCModal()">Help Me Choose</button>
      </v-col>
      <v-col cols="5">
        <v-select :error-messages="errors.includes('request') ? 'Please select a type' : ''"
                  :hide-details="!errors.includes('request')"
                  :items="requestTypeOptions"
                  @change="clearErrors()"
                  filled
                  id="search-type-options-select"
                  v-model="requestAction" />
      </v-col>
      <v-col cols="2">
        <v-select :error-messages="errors.includes('location') ? 'Please select a location' : ''"
                  :hide-details="!errors.includes('location')"
                  :items="locationOptions"
                  filled
                  id="location-options-select"
                  v-model="location" />
      </v-col>
      <v-col cols="5">
        <v-select :error-messages="errors.includes('entity') ? 'Please select a type' : ''"
                  :hide-details="!errors.includes('entity')"
                  :items="entityTypeOptions"
                  @change="clearErrors()"
                  filled
                  id="entity-type-options-select"
                  v-model="entityType" />
      </v-col>
      <NameInput :class="inputCompClass" id="name-input-component" :handleSubmit="handleSubmit" />
      <v-col cols="auto"
             class="my-n9">
        <span id="nr-required-activator"
              class="normal-link"
              @click="activateNRRModal()">Check to see if you need to file a a name request</span>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import Stats from '@/components/new-request/stats'
import newReqModule from '../../store/new-request-module'
import NameInput from './name-input'
import { Component, Vue, Watch } from 'vue-property-decorator'
import { LocationT } from '@/models'

@Component({
  components: { Stats, NameInput }
})
export default class Search extends Vue {
  @Watch('location')
  handleLocation (newVal, oldVal) {
    if (newVal === 'HELP') {
      let type = this.entityType
      newReqModule.mutateLocationInfoModalVisible(true)
      this.$nextTick(function () {
        this.location = oldVal
        this.entityType = type
      })
    }
  }

  @Watch('entityType')
  handleEntityType (newVal, oldVal) {
    if (newVal === 'all' && oldVal !== 'all') {
      newReqModule.mutatePickEntityModalVisible(true)
    }
  }

  get entityType () {
    return newReqModule.entityType
  }
  set entityType (type: string) {
    newReqModule.mutateEntityType(type)
  }
  get entityTypeOptions () {
    return newReqModule.entityTypeOptions
  }
  get errors () {
    return newReqModule.errors
  }
  get inputCompClass () {
    let errorTypes = ['entity', 'request', 'location']
    if (errorTypes.some(type => this.errors.includes(type))) {
      return 'mt-n5'
    }
    return 'mt-n2'
  }
  get location () {
    return newReqModule.location
  }
  set location (location: LocationT) {
    newReqModule.mutateLocation(location)
  }
  get locationOptions () {
    return newReqModule.locationOptions
  }
  get requestAction () {
    return newReqModule.requestAction
  }
  set requestAction (value: string) {
    newReqModule.mutateRequestAction(value)
    if (value === 'all') {
      newReqModule.mutatePickRequestTypeModalVisible(true)
    }
  }
  get requestTypeOptions () {
    return newReqModule.requestTypeOptions
  }

  activateHMCModal () {
    newReqModule.mutateHelpMeChooseModalVisible(true)
  }
  activateNRRModal () {
    newReqModule.mutateNrRequiredModalVisible(true)
  }
  clearErrors () {
    newReqModule.clearErrors()
  }
  handleSubmit (event: Event) {
    event.preventDefault()
    newReqModule.startAnalyzeName()
  }
}

</script>