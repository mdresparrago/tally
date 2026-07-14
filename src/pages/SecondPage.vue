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

            <!-- BULK UPLOAD -->
            <div class="row items-center justify-between q-mb-sm bulk-upload-row">
              <q-btn
                flat dense color="primary" icon="upload_file" label="Bulk upload"
                no-caps :disable="isBatchProcessing"
                @click="triggerBulkUpload"
              />
              <q-btn
                flat dense color="grey-7" icon="download" label="Download template"
                no-caps @click="downloadTemplate"
              />
            </div>
            <input
              ref="fileInput"
              type="file"
              accept=".csv,.xlsx,.xls"
              style="display: none"
              @change="handleBulkFile"
            />

            <q-linear-progress
              v-if="isBatchProcessing"
              :value="batchProgress.total ? batchProgress.current / batchProgress.total : 0"
              color="primary"
              class="q-mb-xs"
            />
            <div v-if="isBatchProcessing" class="text-caption text-grey-7 q-mb-sm">
              Generating receipt {{ batchProgress.current }} of {{ batchProgress.total }}...
            </div>

            <q-separator class="q-my-md" />

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
                    flat dense icon="close" size="sm" color="negative"
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
                {{ formattedDate }}
                <span v-if="form.day && !formattedDate.toUpperCase().includes(form.day.toUpperCase())">
                  ({{ form.day.toUpperCase() }})
                </span>
                <span v-if="form.time">&nbsp;&nbsp;{{ form.time }}</span>
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
              <div class="kv-row text-right-block">
                <div>PAYMENT CUT OFF:</div>
                <div class="mono-bold">
                  {{ form.paymentCutoffDate }}<span v-if="form.paymentCutoffDate && form.paymentCutoffTime">&nbsp;</span>{{ form.paymentCutoffTime }}
                </div>
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
                <div class="mono-bold">{{ (form.courier || '').toUpperCase() }}</div>
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
import { defineComponent, reactive, computed, ref, nextTick } from 'vue'
import { date as qDate, Notify } from 'quasar'
import * as XLSX from 'xlsx'
import JSZip from 'jszip'

export default defineComponent({
  name: 'InvoiceGenerator',

  setup () {
    const receiptOuter = ref(null)
    const fileInput = ref(null)

    const isBatchProcessing = ref(false)
    const batchProgress = reactive({ current: 0, total: 0 })

    const today = new Date()
    const dayOptions = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT']
    const paymentOptions = ['GCASH', 'BANK TRANSFER', 'CASH ON DELIVERY', 'MAYA']

    let idCounter = 1
    const makeItem = () => ({ id: idCounter++, name: '', qty: '', price: '' })

    const form = reactive({
      businessName: 'KAELA COLLECTIVE',
      invoiceNumber: '',
      date: qDate.formatDate(today, 'MM/DD/YYYY'),
      dateISO: qDate.formatDate(today, 'YYYY/MM/DD'),
      time: qDate.formatDate(today, 'HH:mm:ss'),
      day: dayOptions[today.getDay()],
      items: [makeItem(), makeItem()],
      shippingFee: '',
      paymentMode: '',
      paymentCutoffDate: '',
      paymentCutoffTime: '',
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

    const renderReceiptBlob = async () => {
      const html2canvas = (await import('html2canvas')).default

      const canvas = await html2canvas(receiptOuter.value, {
        useCORS: true,
        backgroundColor: null,
        scale: 3,
        imageTimeout: 0,
        logging: false,
        onclone: (clonedDoc) => {
          const clonedCard = clonedDoc.querySelector('.receipt-card')
          if (clonedCard) {
            clonedCard.style.boxShadow = 'none'
            clonedCard.style.webkitBoxShadow = 'none'
          }
        }
      })

      return await new Promise(resolve => canvas.toBlob(resolve, 'image/png'))
    }

    const downloadImage = async () => {
      try {
        const blob = await renderReceiptBlob()

        if (!blob) {
          const html2canvas = (await import('html2canvas')).default
          const canvas = await html2canvas(receiptOuter.value, { useCORS: true, backgroundColor: null, scale: 3 })
          const dataUrl = canvas.toDataURL('image/png')

          const a = document.createElement('a')
          a.href = dataUrl
          a.download = `invoice-${form.invoiceNumber || 'receipt'}.png`
          a.click()
          return
        }

        const fileName = `invoice-${form.invoiceNumber || 'receipt'}.png`
        const file = new File([blob], fileName, { type: 'image/png' })
        const isMobile = /Android|iPhone|iPad|iPod|Mobile/i.test(navigator.userAgent)

        if (
          isMobile &&
          navigator.share &&
          navigator.canShare &&
          navigator.canShare({ files: [file] })
        ) {
          try {
            await navigator.share({ files: [file], title: fileName })
            return
          } catch (err) {
            if (err.name === 'AbortError') return
          }
        }

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
        Notify.create({ type: 'negative', message: 'Unable to generate receipt image.' })
      }
    }

    const triggerBulkUpload = () => {
      fileInput.value?.click()
    }

    const processBatch = async (rows) => {
      isBatchProcessing.value = true
      batchProgress.current = 0
      batchProgress.total = rows.length

      const zip = new JSZip()

      try {
        for (let i = 0; i < rows.length; i++) {
          applyRowToForm(rows[i])

          await nextTick()
          await new Promise(resolve => setTimeout(resolve, 60))

          const blob = await renderReceiptBlob()
          if (blob) {
            const fileName = `invoice-${form.invoiceNumber || (i + 1)}.png`
            zip.file(fileName, blob)
          }

          batchProgress.current = i + 1
        }

        const zipBlob = await zip.generateAsync({ type: 'blob' })
        const url = URL.createObjectURL(zipBlob)
        const a = document.createElement('a')
        a.href = url
        a.download = `receipts-batch-${Date.now()}.zip`
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        setTimeout(() => URL.revokeObjectURL(url), 1000)

        Notify.create({ type: 'positive', message: `Generated ${rows.length} receipts.` })
      } catch (err) {
        console.error(err)
        Notify.create({ type: 'negative', message: 'Batch generation failed partway through.' })
      } finally {
        isBatchProcessing.value = false
      }
    }

    const handleBulkFile = async (event) => {
      const file = event.target.files[0]
      event.target.value = ''
      if (!file) return

      try {
        const data = await new Promise((resolve, reject) => {
          const reader = new FileReader()
          reader.onload = (e) => {
            try {
              const arr = new Uint8Array(e.target.result)
              const workbook = XLSX.read(arr, { type: 'array' })
              const sheet = workbook.Sheets[workbook.SheetNames[0]]
              const json = XLSX.utils.sheet_to_json(sheet, { defval: '', raw: false })
              resolve(json)
            } catch (err) { reject(err) }
          }
          reader.onerror = reject
          reader.readAsArrayBuffer(file)
        })

        if (!data.length) {
          Notify.create({ type: 'warning', message: 'No orders found in that file.' })
          return
        }

        const groupedOrders = []
        const map = new Map()

        data.forEach(row => {
          const invNum = row.invoiceNumber !== undefined ? String(row.invoiceNumber).trim() : ''

          if (!map.has(invNum)) {
            const baseOrder = {
              ...row,
              invoiceNumber: invNum,
              items: []
            }

            if (row.name && String(row.name).trim()) {
              baseOrder.items.push({
                id: idCounter++,
                name: String(row.name).trim(),
                qty: Number(row.qty) || 1,
                price: Number(row.price) || 0
              })
            }

            map.set(invNum, baseOrder)
            groupedOrders.push(baseOrder)
          } else {
            const existingOrder = map.get(invNum)

            if (row.date) existingOrder.date = row.date
            if (row.time) existingOrder.time = row.time
            if (row.day) existingOrder.day = row.day
            if (row.shippingFee !== undefined && row.shippingFee !== '') existingOrder.shippingFee = row.shippingFee
            if (row.paymentMode) existingOrder.paymentMode = row.paymentMode

            // Unnamed/Shifted column fallbacks
            const rawCutoffDate = row.paymentCutoffDate || row.__EMPTY_1 || row.paymentCut
            if (rawCutoffDate) existingOrder.paymentCutoffDate = rawCutoffDate

            const rawCutoffTime = row.paymentCutoffTime || row.paymentCu
            if (rawCutoffTime) existingOrder.paymentCutoffTime = rawCutoffTime

            if (row.packingDate) existingOrder.packingDate = row.packingDate
            if (row.shippingDate) existingOrder.shippingDate = row.shippingDate
            if (row.courier) existingOrder.courier = row.courier

            if (row.name && String(row.name).trim()) {
              existingOrder.items.push({
                id: idCounter++,
                name: String(row.name).trim(),
                qty: Number(row.qty) || 1,
                price: Number(row.price) || 0
              })
            }
          }
        })

        await processBatch(groupedOrders)
      } catch (err) {
        console.error(err)
        Notify.create({ type: 'negative', message: 'Could not read that file.' })
      }
    }

    const applyRowToForm = (order) => {
      const formatDateValue = (val) => {
        if (!val) return ''
        return String(val).trim()
      }

      if (order.businessName) form.businessName = String(order.businessName)
      form.invoiceNumber = order.invoiceNumber

      form.date = formatDateValue(order.date)
      form.time = order.time ? String(order.time) : ''
      form.day = order.day ? String(order.day).toUpperCase() : ''

      form.items = order.items.length ? order.items : [makeItem()]
      form.shippingFee = (order.shippingFee !== undefined && order.shippingFee !== '') ? Number(order.shippingFee) : 0
      form.paymentMode = order.paymentMode ? String(order.paymentMode).toUpperCase() : ''

      const rawCutoff = order.paymentCutoffDate || order.__EMPTY_1 || order.paymentCut
      form.paymentCutoffDate = formatDateValue(rawCutoff)

      const rawCutoffTime = order.paymentCutoffTime || order.paymentCu
      form.paymentCutoffTime = rawCutoffTime ? String(rawCutoffTime) : ''

      form.packingDate = formatDateValue(order.packingDate)
      form.shippingDate = formatDateValue(order.shippingDate)
      form.courier = order.courier ? String(order.courier) : ''
    }

    const downloadTemplate = () => {
      const headers = [
        'invoiceNumber', 'date', 'time', 'day',
        'name', 'qty', 'price',
        'shippingFee', 'paymentMode', 'paymentCutoffDate', 'paymentCutoffTime',
        'packingDate', 'shippingDate', 'courier'
      ]
      const sample = [{
        invoiceNumber: 'KC-260627-001',
        date: qDate.formatDate(today, 'MM/DD/YYYY'),
        time: '15:30:00',
        day: dayOptions[today.getDay()],
        name: 'Backless ruffle hem halter dress',
        qty: 1,
        price: 299,
        shippingFee: 65,
        paymentMode: 'MAYA',
        paymentCutoffDate: '07/20/2026',
        paymentCutoffTime: '23:59',
        packingDate: '07/26/2026',
        shippingDate: '07/26/2026',
        courier: 'ANGKAS'
      }]

      const ws = XLSX.utils.json_to_sheet(sample, { header: headers })
      const wb = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(wb, ws, 'Orders')
      XLSX.writeFile(wb, 'receipt-order-template.xlsx')
    }

    return {
      receiptOuter,
      fileInput,
      isBatchProcessing,
      batchProgress,
      form,
      dayOptions,
      paymentOptions,
      addItem,
      removeItem,
      totalAmount,
      formatNum,
      formattedDate,
      onDatePicked,
      downloadImage,
      triggerBulkUpload,
      handleBulkFile,
      downloadTemplate
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

  .bulk-upload-row {
    flex-wrap: wrap;
  }

  .preview-wrap {
    position: sticky;
    top: 16px;
    display: flex;
    justify-content: center;
    width: 100%;
  }

  .receipt-outer {
    background-image: url('../assets/maroon-bg.jpg');
    width: 100%;
    max-width: 450px;
    background-position: center;
    border-radius: 10px;
    padding: 24px;
    box-sizing: border-box;
    display: flex;
    justify-content: center;
  }

  .receipt-card {
    background-image: url('../assets/paper-bg.jpg');
    background-size: cover;
    background-position: center;
    width: 100%;
    max-width: 450px;
    padding: 28px;
    color: #3a1030;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 13px;
    line-height: 1.5;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
    box-sizing: border-box;

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
    white-space: pre-wrap;
    word-break: break-word;
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
    align-items: flex-start;
    gap: 12px;
    margin-bottom: 4px;

    &.text-right-block {
      .mono-bold {
        text-align: right;
        max-width: 60%;
        word-break: break-word;
      }
    }
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
