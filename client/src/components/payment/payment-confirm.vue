<template>
  <v-card>
    <!--<header class="font-weight-bold px-3 py-3">
      <slot name="header">Payment Details</slot>
    </header>-->

    <div v-show="fetchError">
      <v-alert color="error" icon="warning" outlined>{{fetchError}}</v-alert>
    </div>

    <ul class="fee-list" v-show="!fetchError">
      <li class="container fee-list__item">
        <div class="fee-list__item-name">
          NR Number<sup>*</sup>
        </div>
        <div class="fee-list__item-value"><strong>{{nrNum}}</strong></div>
      </li>
      <li>
        <small style="font-weight: normal; font-size: 0.7rem">
          * Use this code to check the status of your application
        </small>
      </li>
    </ul>

    <request-details
      v-bind:applicant="applicant"
      v-bind:nameChoices="nameChoices"
      v-bind:name="name"
    />

    <ul class="fee-list" v-show="!fetchError">
      <!-- <li class="container fee-list__item" v-if="invoice">
        <div class="fee-list__item-name">Payment Ref #</div>
        <div class="fee-list__item-value">{{invoice.references[0].reference_number}}</div>
      </li> -->
      <li class="container fee-list__item" v-if="summary">
        <div class="fee-list__item-name">Payment Date</div>
        <div class="fee-list__item-value">{{new Date(summary.completionDate).toLocaleDateString("en-US")}}</div>
      </li>
      <li class="container fee-list__item" v-if="invoice">
        <div class="fee-list__item-name">Amount</div>
        <div class="fee-list__item-value">${{invoice.total.toFixed(2)}} CAD</div>
      </li>
      <li class="container fee-list__item" v-if="summary">
        <div class='fee-list__item-name'>Status</div>
        <div class='fee-list__item-value'>{{summary.statusCode}}</div>
      </li>
    </ul>
  </v-card>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'
import '../../plugins/vuetify'

import RequestDetails from '@/components/common/request-details.vue'
import { ApplicantI } from '@/models'

@Component({
  components: {
    RequestDetails
  }
})
export default class PaymentConfirm extends Vue {
  @Prop(String) nrNum: string
  @Prop(Object) summary: any
  @Prop(Object) invoice: any
  @Prop(Object) applicant: ApplicantI
  @Prop(Array) nameChoices: {
    type: any[]
    required: false
  }
  @Prop(String) name: string
  // @Prop(Boolean) priorityRequest: boolean
  // @Prop(Object) payment: any
  @Prop({ default: {
    invoice: {
      created_on: '',
      total: 0.00,
      references: [{
        invoice_number: '',
        status_code: ''
      }]
    },
    summary: {
      completionDate: '',
      statusCode: ''
    }
  } })

  protected fetchError: string = ''
}
</script>

<style lang="scss" scoped>
@import "../../assets/scss/theme";

header {
  color: #fff;
  background: $BCgovBlue5;
}

.container {
  display: flex;
  flex-flow: row nowrap;
  line-height: 1.2rem;
  font-size: 0.875rem;
}

.filing_invoice-list {
  border-bottom: 1px solid $gray3;
}

.filing_invoice-list__item {
  &-name, &-value {
    font-weight: 700;
  }

  &-name {
    flex: 1 1 auto;
    margin-right: 2rem;
  }

  &-value {
    flex: 0 0 auto;
    text-align: right;
  }
}

.filing_invoice-list__item + .filing_invoice-list__item {
  border-top: 1px solid $gray3;
}

.filing_invoice-total {
  align-items: center;
  letter-spacing: -0.01rem;
  line-height: auto;

  &__name {
    flex: 1 1 auto;
    margin-right: auto;
    font-weight: 700;
  }

  &__currency {
    margin-right: 0.5rem;
    color: $gray5;
    font-weight: 500;
  }

  &__value {
    font-size: 1.65rem;
    font-weight: 700;
  }
}

.container.fee-total {
  font-weight: bold;
}

.fee-list__item-name {
  font-weight: bold;
}

.container.fee-list__item {
  justify-content: space-between;
}

.container.fee-list__item {
  border-bottom: 1px dotted grey;
}
</style>
