<template>
  <v-card class='mb-5' v-if="summary">
    <header class='font-weight-bold px-3 py-3'>
      <slot name='header'>
        <span>
          <small>{{new Date(summary.completionDate).toLocaleDateString('en-US')}}</small>
        </span>
        <span style="float: right" v-if="invoice">
          <small><small>Invoice No.</small></small> {{invoice.references[0].invoice_number}}
        </span>
      </slot>
    </header>

    <div v-show='fetchError'>
      <v-alert color='error' icon='warning' outlined>{{fetchError}}</v-alert>
    </div>

    <ul class='fee-list' v-show='!fetchError'>
      <li class='container fee-list__item' v-if="invoice">
        <div class='fee-list__item-name'>Amount</div>
        <div class='fee-list__item-value'>${{invoice.total.toFixed(2)}} CAD</div>
      </li>
      <li class='container fee-list__item' v-if="summary">
        <div class='fee-list__item-name'>Status</div>
        <div class='fee-list__item-value'>{{summary.statusCode}}</div>
      </li>
      <li class='container fee-list__item'>
        <div class='fee-list__item-name'>Receipt</div>
        <div class='fee-list__item-value'>
          <v-btn @click='downloadReceipt' id='download-receipt-btn' class='primary' text small>Download PDF</v-btn>
        </div>
      </li>
    </ul>

  </v-card>
</template>

<script lang='ts'>
import { Component, Mixins, Prop } from 'vue-property-decorator'
import '../../plugins/vuetify'

import { NameRequestPaymentResponse } from '@/modules/payment/models'

import RequestDetails from '@/components/common/request-details.vue'
import PaymentMixin from '@/components/payment/payment-mixin'

@Component({
  components: {
    RequestDetails
  }
})
export default class PaymentSummary extends Mixins(PaymentMixin) {
  @Prop(Number) id: number
  @Prop(Object) summary: any
  @Prop(Object) invoice: any
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

  async downloadReceipt () {
    const { id } = this
    await this.fetchReceiptPdf(id)
  }
}
</script>

<style lang='scss' scoped>
@import '../../assets/scss/theme';

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
