<template>
  <div class="invoice-generator">
    <div class="row q-col-gutter-md">
      <!-- LEFT: FORM -->
      <div class="col-xs-12 col-sm-7 col-md-8">
        <q-card flat bordered class="form-card q-ma-md">
          <q-card-section>
            <div class="form-header">

                <div>
                  <div class="text-h5 text-weight-bold">
                    Invoice Generator
                  </div>

                  <div class="text-grey-7 q-pb-md">
                    Complete the information and preview updates instantly.
                  </div>
                </div>
              </div>

            <q-input
              v-model="form.businessName"
              label="Business Name"
              dense outlined class="q-mb-sm"
            />

            <div class="row q-col-gutter-sm">
              <div class="col-6">
                <q-input
                  v-model="form.invoiceNumber"
                  label="Invoice #"
                  dense outlined class="q-mb-sm"
                  prefix="#"
                />
              </div>
              <div class="col-6">
                <q-input
                  v-model="form.date"
                  label="Date"
                  dense outlined class="q-mb-sm"
                  mask="##/##/####"
                  placeholder="MM/DD/YYYY"
                >
                  <template #append>
                    <q-icon name="event" class="cursor-pointer">
                      <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                        <q-date v-model="form.dateISO" mask="MM/DD/YYYY" @update:model-value="onDatePicked">
                          <div class="row items-center justify-end">
                            <q-btn v-close-popup label="Close" color="primary" flat />
                          </div>
                        </q-date>
                      </q-popup-proxy>
                    </q-icon>
                  </template>
                </q-input>
              </div>
            </div>

            <div class="row q-col-gutter-sm">
              <div class="col-6">
                <q-input v-model="form.time" label="Time" dense outlined class="q-mb-sm" placeholder="HH:MM:SS" />
              </div>
              <div class="col-6">
                <q-select
                  v-model="form.day"
                  :options="dayOptions"
                  label="Day"
                  dense outlined class="q-mb-sm"
                />
              </div>
            </div>

            <q-separator class="q-my-md" />

            <div class="row items-center justify-between q-mb-sm">
              <div class="text-subtitle2">Items</div>
              <q-btn
                flat dense color="primary" icon="add" label="Add Product"
                @click="addItem"
              />
            </div>

            <div
              v-for="(item, idx) in form.items"
              :key="item.id"
              class="item-row q-mb-sm"
            >
              <div class="row q-col-gutter-sm items-start">
                <div class="col-6">
                  <q-input
                    v-model="item.name"
                    label="Item name"
                    dense outlined type="textarea" autogrow
                  />
                </div>
                <div class="col-3">
                  <q-input
                    v-model.number="item.qty"
                    label="Qty"
                    dense outlined type="number" min="1"
                  />
                </div>
                <div class="col-2">
                  <q-input
                    v-model.number="item.price"
                    label="Price"
                    dense outlined type="number" min="0" step="0.01"
                  />
                </div>
                <div class="col-1 flex flex-center">
                  <q-btn
                    flat dense  icon="close" size="sm" color="negative"
                    :disable="form.items.length === 1"
                    @click="removeItem(idx)"
                  />
                </div>
              </div>
            </div>

            <q-separator class="q-my-md" />

            <q-input
              v-model.number="form.shippingFee"
              label="Shipping Fee"
              dense outlined type="number" min="0" step="0.01" class="q-mb-sm"
              prefix="₱"
            />

            <q-select
              v-model="form.paymentMode"
              :options="paymentOptions"
              label="Mode of Payment"
              dense outlined class="q-mb-sm"
            />

            <div class="row q-col-gutter-sm">
              <div class="col-6">
                <q-input v-model="form.paymentCutoffDate" label="Payment Cutoff Date" dense outlined class="q-mb-sm" placeholder="MM/DD/YYYY" />
              </div>
              <div class="col-6">
                <q-input v-model="form.paymentCutoffTime" label="Cutoff Time" dense outlined class="q-mb-sm" placeholder="12:00 NN" />
              </div>
            </div>

            <q-separator class="q-my-md" />

            <div class="row q-col-gutter-sm">
              <div class="col-6">
                <q-input v-model="form.packingDate" label="Packing Date" dense outlined class="q-mb-sm" placeholder="MM/DD/YYYY" />
              </div>
              <div class="col-6">
                <q-input v-model="form.shippingDate" label="Shipping Date" dense outlined class="q-mb-sm" placeholder="MM/DD/YYYY" />
              </div>
            </div>

            <q-input
              v-model="form.courier"
              label="Courier"
              dense outlined class="q-mb-sm"
            />

            <q-separator class="q-my-md" />

            <div class="text-subtitle2 q-mb-sm">Footer Note</div>
            <q-input
              v-model="form.footerNote"
              type="textarea"
              autogrow
              dense outlined
            />
          </q-card-section>
        </q-card>
          <div class="preview-actions q-ma-sm flex justify-end">
            <q-btn
              color="primary" rounded icon="download" label="Download as Image"
              @click="downloadImage" no-caps unelevated
            />
          </div>

      </div>

      <!-- RIGHT: RECEIPT PREVIEW -->
      <div class="col-xs-12 col-sm-5 col-md-4">
        <div class="preview-wrap">
          <div class="receipt-outer" ref="receiptOuter">
            <div class="receipt-card">
              <div class="receipt-header row items-center justify-between">
                <div class="mono-bold">INVOICE</div>
                <div class="mono-bold">#{{ form.invoiceNumber || '000000000' }}</div>
              </div>

              <div class="logo-wrap">
                <img src="../assets/logo.png" alt="logo" class="logo-img" />
              </div>

              <div class="business-name">{{ form.businessName }}</div>

              <div class="dashed-line" />

              <div class="datetime-row">
                {{ formattedDate }} ({{ (form.day || '').toUpperCase() }})&nbsp;&nbsp;{{ form.time }}
              </div>

              <div class="dashed-line" />

              <div class="items-block">
                <div v-for="item in form.items" :key="item.id" class="item-line">
                  <div class="item-name">
                    {{ (item.name || '').toUpperCase() }} ({{ item.qty || 0 }})
                  </div>
                  <div class="item-price">{{ formatNum(item.qty * item.price) }}</div>
                </div>

                <div class="item-line">
                  <div class="item-name">SHIPPING FEE</div>
                  <div class="item-price">{{ formatNum(form.shippingFee) }}</div>
                </div>
              </div>

              <div class="total-line">
                <div class="mono-bold">TOTAL AMOUNT DUE</div>
                <div class="mono-bold">{{ formatNum(totalAmount) }}</div>
              </div>

              <div class="dashed-line" />

              <div class="kv-row">
                <div>MODE OF PAYMENT:</div>
                <div class="mono-bold">{{ (form.paymentMode || '').toUpperCase() }}</div>
              </div>
              <div class="kv-row">
                <div>PAYMENT CUT OFF:</div>
                <div class="mono-bold">{{ form.paymentCutoffDate }} {{ form.paymentCutoffTime }}</div>
              </div>

              <div class="dashed-line" />

              <div class="kv-row">
                <div>PACKING:</div>
                <div class="mono-bold">{{ form.packingDate }}</div>
              </div>
              <div class="kv-row">
                <div>SHIPPING:</div>
                <div class="mono-bold">{{ form.shippingDate }}</div>
              </div>
              <div class="kv-row">
                <div>COURIER:</div>
                <div class="mono-bold">{{ form.courier }}</div>
              </div>

              <div class="footer-note">
                {{ form.footerNote }}
              </div>
            </div>
          </div>
        </div>
        <div class="q-pa-md q-ma-md flex justify-center text-grey-5">
        Tally by dani
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent, reactive, computed, ref } from 'vue'
import { date as qDate, Notify } from 'quasar'

export default defineComponent({
  name: 'InvoiceGenerator',

  setup () {
    const receiptOuter = ref(null)

    const today = new Date()
    const dayOptions = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT']
    const paymentOptions = ['GCASH', 'BANK TRANSFER', 'CASH ON DELIVERY', 'MAYA']

    let idCounter = 1
    const makeItem = () => ({ id: idCounter++, name: '', qty: 1, price: 0 })

    const form = reactive({
      businessName: 'KAELA COLLECTIVE',
      invoiceNumber: '',
      date: qDate.formatDate(today, 'MM/DD/YYYY'),
      dateISO: qDate.formatDate(today, 'YYYY/MM/DD'),
      time: qDate.formatDate(today, 'HH:mm:ss'),
      day: dayOptions[today.getDay()],
      items: [makeItem(), makeItem()],
      shippingFee: 65,
      paymentMode: 'GCASH',
      paymentCutoffDate: '',
      paymentCutoffTime: '12:00 NN',
      packingDate: '',
      shippingDate: '',
      courier: '',
      footerNote:
        'PLEASE SEND A SCREENSHOT OF YOUR RECEIPT ONCE SETTLED.\n' +
        'NO PAYMENT, NO SHIPPING. ORDERS WILL ONLY BE PROCESSED ONCE PAYMENT IS CONFIRMED.\n' +
        'KINDLY DOUBLE-CHECK YOUR INVOICE BEFORE SENDING PAYMENT.\n' +
        'THANK YOU!'
    })

    const onDatePicked = (val) => {
      // val comes back as MM/DD/YYYY because of the mask prop above
      form.date = val
    }

    const addItem = () => form.items.push(makeItem())
    const removeItem = (idx) => {
      if (form.items.length > 1) form.items.splice(idx, 1)
    }

    const itemsTotal = computed(() =>
      form.items.reduce((sum, it) => sum + (Number(it.qty) || 0) * (Number(it.price) || 0), 0)
    )

    const totalAmount = computed(() => itemsTotal.value + (Number(form.shippingFee) || 0))

    const formatNum = (n) =>
      (Number(n) || 0).toLocaleString('en-PH', { minimumFractionDigits: 2, maximumFractionDigits: 2 })

    const formattedDate = computed(() => form.date)

const downloadImage = async () => {
  try {
    const html2canvas = (await import('html2canvas')).default

    // const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent)

    const canvas = await html2canvas(receiptOuter.value, {
      useCORS: true,
      backgroundColor: null,
      scale: 3,
      // FIX FOR MOBILE DARKENING:
      imageTimeout: 0,
      logging: false,
      onclone: (clonedDoc) => {
        // Find the receipt card inside the screenshot factory
        const clonedCard = clonedDoc.querySelector('.receipt-card')
        if (clonedCard) {
          // Remove the shadow completely so mobile doesn't bleed it across the paper texture
          clonedCard.style.boxShadow = 'none'
          clonedCard.style.webkitBoxShadow = 'none'
        }
      }
    })

    let blob = await new Promise(resolve =>
      canvas.toBlob(resolve, 'image/png')
    )

    // Safari fallback
    if (!blob) {
      const dataUrl = canvas.toDataURL('image/png')

      const a = document.createElement('a')
      a.href = dataUrl
      a.download = `invoice-${form.invoiceNumber || 'receipt'}.png`
      a.click()

      return
    }

    const fileName = `invoice-${form.invoiceNumber || 'receipt'}.png`
    const file = new File([blob], fileName, {
      type: 'image/png'
    })

    // iPhone / Android Share Sheet
    const isMobile =
  /Android|iPhone|iPad|iPod|Mobile/i.test(navigator.userAgent)

if (
  isMobile &&
  navigator.share &&
  navigator.canShare &&
  navigator.canShare({ files: [file] })
) {
  try {
    await navigator.share({
      files: [file],
      title: fileName
    })
    return
  } catch (err) {
    if (err.name === 'AbortError') return
  }
    }

    // Desktop download
    const url = URL.createObjectURL(blob)

    const a = document.createElement('a')
    a.href = url
    a.download = fileName

    document.body.appendChild(a)
    a.click()
    document.body.removeChild(a)

    setTimeout(() => URL.revokeObjectURL(url), 1000)

  } catch (err) {
    console.error(err)

    Notify.create({
      type: 'negative',
      message: 'Unable to generate receipt image.'
    })
  }
}

    return {
      receiptOuter,
      form,
      dayOptions,
      paymentOptions,
      addItem,
      removeItem,
      totalAmount,
      formatNum,
      formattedDate,
      onDatePicked,
      downloadImage
    }
  }
})
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600;700&display=swap');

.invoice-generator {
  font-family: 'IBM Plex Mono', monospace;

  .form-card {
    font-family: 'IBM Plex Mono', monospace;
    border-radius: 15px;
  }

  .form-title {
    font-family: 'IBM Plex Mono', monospace;
    font-weight: 600;
    margin-bottom: 12px;
  }

  .preview-wrap {
    position: sticky;
    top: 16px;
    display: flex;
    justify-content: center;
    width: 100%;
  }

  // Outer maroon background frame
  .receipt-outer {
    background-image: url('../assets/maroon-bg.jpg');
    width: 100%;
    max-width: 450px;
    background-position: center;
    border-radius: 10px;
    padding: 24px; // Slightly reduced padding so it scales nicely on smaller viewports
    box-sizing: border-box;
    display: flex;
    justify-content: center;
  }

  // Cream receipt card
  .receipt-card {
    background-image: url('../assets/paper-bg.jpg');
    background-size: cover;
    background-position: center;
    width: 100%; // Changed from fit-content to 100% so it stays stable
    max-width: 450px; // OPTIONAL: Keeps the receipt from looking ridiculously wide on massive desktop screens
    padding: 28px;
    color: #3a1030;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 13px;
    line-height: 1.5;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
    box-sizing: border-box;

    // jagged torn-paper edges top & bottom
    clip-path: polygon(
      0% 1%, 3% 0%, 6% 1.2%, 9% 0%, 12% 1.2%, 15% 0%, 18% 1.2%, 21% 0%,
      24% 1.2%, 27% 0%, 30% 1.2%, 33% 0%, 36% 1.2%, 39% 0%, 42% 1.2%, 45% 0%,
      48% 1.2%, 51% 0%, 54% 1.2%, 57% 0%, 60% 1.2%, 63% 0%, 66% 1.2%, 69% 0%,
      72% 1.2%, 75% 0%, 78% 1.2%, 81% 0%, 84% 1.2%, 87% 0%, 90% 1.2%, 93% 0%,
      96% 1.2%, 100% 0%,
      100% 99%, 97% 100%, 94% 98.8%, 91% 100%, 88% 98.8%, 85% 100%, 82% 98.8%,
      79% 100%, 76% 98.8%, 73% 100%, 70% 98.8%, 67% 100%, 64% 98.8%, 61% 100%,
      58% 98.8%, 55% 100%, 52% 98.8%, 49% 100%, 46% 98.8%, 43% 100%, 40% 98.8%,
      37% 100%, 34% 98.8%, 31% 100%, 28% 98.8%, 25% 100%, 22% 98.8%, 19% 100%,
      16% 98.8%, 13% 100%, 10% 98.8%, 7% 100%, 4% 98.8%, 0% 100%
    );
  }
  .mono-bold {
    font-weight: 700;
  }

  .logo-wrap {
    display: flex;
    justify-content: center;
    margin: 8px 0 4px;
  }

  .logo-img {
    width: 110px;
    height: 110px;
    object-fit: contain;
  }

  .business-name {
    text-align: center;
    font-weight: 700;
    letter-spacing: 1px;
    margin-bottom: 14px;
  }

  .dashed-line {
    border-top: 1px dashed #3a1030;
    margin: 10px 0;
    opacity: 0.6;
  }

  .datetime-row {
    text-align: center;
    font-weight: 500;
  }

  .items-block {
    margin: 6px 0;
  }

  .item-line {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 12px;
    margin-bottom: 10px;
  }

  .item-name {
    flex: 1;
    white-space: pre-wrap;
  }

  .item-price {
    white-space: nowrap;
  }

  .total-line {
    display: flex;
    justify-content: space-between;
    margin-top: 6px;
  }

  .kv-row {
    display: flex;
    justify-content: space-between;
    gap: 12px;
    margin-bottom: 4px;
  }

  .footer-note {
    margin-top: 16px;
    text-align: center;
    font-weight: 700;
    font-size: 11px;
    white-space: pre-line;
    line-height: 1.6;
  }

  .item-row {
    background: rgba(0, 0, 0, 0.02);
    border-radius: 6px;
    padding: 6px;
  }
}
</style>
