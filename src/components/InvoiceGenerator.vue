<template>
  <div class="invoice-generator">
    <header>
      <h1>Online Invoice Generator</h1>
    </header>

    <div class="container">
      <div class="controls">
        <button @click="generatePDF" class="btn primary">Download PDF</button>
        <div class="currency-selector">
          <label for="currency">Currency:</label>
          <select id="currency" v-model="invoice.currency">
            <option value="USD">USD ($)</option>
            <option value="USD">INR (₹)</option>
            <option value="EUR">EUR (€)</option>
            <option value="GBP">GBP (£)</option>
            <option value="JPY">JPY (¥)</option>
            <option value="CAD">CAD ($)</option>
            <option value="AUD">AUD ($)</option>
          </select>
        </div>
      </div>

      <div class="invoice-form-container">
        <div class="invoice-form">
          <!-- Company Details Section -->
          <section class="form-section">
            <h2>Your Company</h2>
            <div class="logo-upload" @dragover.prevent @drop.prevent="onDropLogo">
              <div v-if="!invoice.companyLogo" class="logo-placeholder">
                <span class="upload-icon">+</span>
                <p>Drag & drop your logo here or</p>
                <label for="logo-input" class="upload-btn">Browse Files</label>
                <input 
                  type="file" 
                  id="logo-input" 
                  accept="image/*" 
                  @change="onSelectLogo" 
                  class="hidden-input"
                />
              </div>
              <div v-else class="logo-preview">
                <img :src="invoice.companyLogo" alt="Company Logo" />
                <button @click="removeLogo" class="remove-logo">×</button>
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label for="companyName">Company Name</label>
                <input type="text" id="companyName" v-model="invoice.companyName" placeholder="Your company name" />
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label for="companyAddress">Address</label>
                <textarea id="companyAddress" v-model="invoice.companyAddress" placeholder="Your company address"></textarea>
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label for="companyEmail">Email</label>
                <input type="email" id="companyEmail" v-model="invoice.companyEmail" placeholder="comany@example.com" />
              </div>
              <div class="form-group">
                <label for="companyPhone">Phone</label>
                <input type="tel" id="companyPhone" v-model="invoice.companyPhone" placeholder="(123) 456-7890"/>
              </div>
            </div>
          </section>

          <!-- Client Details Section -->
          <section class="form-section">
            <h2>Client Information</h2>
            <div class="form-row">
              <div class="form-group">
                <label for="clientName">Client Name</label>
                <input type="text" id="clientName" v-model="invoice.clientName" placeholder="Client name"/>
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label for="clientAddress">Address</label>
                <textarea id="clientAddress" v-model="invoice.clientAddress" placeholder="Client address"></textarea>
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label for="clientEmail">Email</label>
                <input type="email" id="clientEmail" v-model="invoice.clientEmail" placeholder="comany@example.com"/>
              </div>
              <div class="form-group">
                <label for="clientPhone">Phone</label>
                <input type="tel" id="clientPhone" v-model="invoice.clientPhone" placeholder="(123) 456-7890"/>
              </div>
            </div>
          </section>

          <!-- Invoice Details Section -->
          <section class="form-section">
            <h2>Invoice Details</h2>
            <div class="form-row">
              <div class="form-group">
                <label for="invoiceNumber">Invoice Number</label>
                <input type="text" id="invoiceNumber" v-model="invoice.invoiceNumber" />
              </div>
              <div class="form-group">
                <label for="invoiceDate">Date</label>
                <input type="date" id="invoiceDate" v-model="invoice.invoiceDate" />
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label for="dueDate">Due Date</label>
                <input type="date" id="dueDate" v-model="invoice.dueDate" />
              </div>
            </div>
          </section>

          <!-- Items Section -->
          <section class="form-section">
            <h2>Items</h2>
            <div class="items-table">
              <div class="items-header">
                <div class="item-description">Description</div>
                <div class="item-quantity">Quantity</div>
                <div class="item-price">Price</div>
                <div class="item-total">Total</div>
                <div class="item-actions"></div>
              </div>
              <div v-for="(item, index) in invoice.items" :key="index" class="item-row">
                <div class="item-description">
                  <input type="text" v-model="item.description" placeholder="Item description" />
                </div>
                <div class="item-quantity">
                  <input type="number" v-model.number="item.quantity" min="1" @input="calculateTotals" />
                </div>
                <div class="item-price">
                  <input type="number" v-model.number="item.price" min="0" step="0.01" @input="calculateTotals" />
                </div>
                <div class="item-total">
                  {{ getCurrencySymbol(invoice.currency) }}{{ calculateItemTotal(item).toFixed(2) }}
                </div>
                <div class="item-actions">
                  <button @click="removeItem(index)" class="btn-remove">×</button>
                </div>
              </div>
            </div>
            <button @click="addItem" class="btn add-item">+ Add Item</button>
          </section>

          <!-- Totals Section -->
          <section class="form-section">
            <h2>Totals</h2>
            <div class="totals-container">
              <div class="form-row">
                <div class="form-group">
                  <label for="taxRate">Tax Rate (%)</label>
                  <input type="number" id="taxRate" v-model.number="invoice.taxRate" min="0" max="100" step="0.01" @input="calculateTotals" />
                </div>
                <div class="form-group">
                  <label for="discount">Discount (%)</label>
                  <input type="number" id="discount" v-model.number="invoice.discount" min="0" max="100" step="0.01" @input="calculateTotals" />
                </div>
              </div>
              <div class="totals-summary">
                <div class="total-row">
                  <span>Subtotal:</span>
                  <span>{{ getCurrencySymbol(invoice.currency) }}{{ invoice.subtotal.toFixed(2) }}</span>
                </div>
                <div class="total-row">
                  <span>Discount ({{ invoice.discount }}%):</span>
                  <span>-{{ getCurrencySymbol(invoice.currency) }}{{ invoice.discountAmount.toFixed(2) }}</span>
                </div>
                <div class="total-row">
                  <span>Tax ({{ invoice.taxRate }}%):</span>
                  <span>{{ getCurrencySymbol(invoice.currency) }}{{ invoice.taxAmount.toFixed(2) }}</span>
                </div>
                <div class="total-row grand-total">
                  <span>Total:</span>
                  <span>{{ getCurrencySymbol(invoice.currency) }}{{ invoice.total.toFixed(2) }}</span>
                </div>
              </div>
            </div>
          </section>

          <!-- Notes Section -->
          <section class="form-section">
            <h2>Notes</h2>
            <div class="form-group">
              <textarea v-model="invoice.notes" placeholder="Add any additional notes here..."></textarea>
            </div>
          </section>
        </div>

        <!-- Invoice Preview -->
        <div class="invoice-preview">
          <h2>Preview</h2>
          <div class="preview-container" ref="previewContainer">
            <div class="preview-header">
              <div class="preview-logo" v-if="invoice.companyLogo">
                <img :src="invoice.companyLogo" alt="Company Logo" />
              </div>
              <div class="preview-company">
                <h3>{{ invoice.companyName || 'Your Company Name' }}</h3>
                <p>{{ invoice.companyAddress || 'Company Address' }}</p>
                <p>{{ invoice.companyEmail || 'company@example.com' }} | {{ invoice.companyPhone || '(123) 456-7890' }}</p>
              </div>
            </div>
            
            <div class="preview-invoice-details">
              <h2>INVOICE</h2>
              <div class="invoice-info">
                <div>
                  <p><strong>Invoice #:</strong> {{ invoice.invoiceNumber || 'INV-001' }}</p>
                  <p><strong>Date:</strong> {{ formatDate(invoice.invoiceDate) }}</p>
                  <p><strong>Due Date:</strong> {{ formatDate(invoice.dueDate) }}</p>
                </div>
              </div>
            </div>
            
            <div class="preview-client">
              <h4>Bill To:</h4>
              <p><strong>{{ invoice.clientName || 'Client Name' }}</strong></p>
              <p>{{ invoice.clientAddress || 'Client Address' }}</p>
              <p>{{ invoice.clientEmail || 'client@example.com' }} | {{ invoice.clientPhone || '(123) 456-7890' }}</p>
            </div>
            
            <div class="preview-items">
              <table>
                <thead>
                  <tr>
                    <th>Description</th>
                    <th>Quantity</th>
                    <th>Price</th>
                    <th>Total</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, index) in invoice.items" :key="index">
                    <td>{{ item.description || 'Item description' }}</td>
                    <td>{{ item.quantity }}</td>
                    <td>{{ getCurrencySymbol(invoice.currency) }}{{ calculateItemTotal(item).toFixed(2) }}</td>
                    <td>{{ getCurrencySymbol(invoice.currency) }}{{ calculateItemTotal(item).toFixed(2) }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
            
            <div class="preview-totals">
              <div class="total-line">
                <span>Subtotal:</span>
                <span>{{ getCurrencySymbol(invoice.currency) }}{{ invoice.subtotal.toFixed(2) }}</span>
              </div>
              <div class="total-line" v-if="invoice.discount > 0">
                <span>Discount ({{ invoice.discount }}%):</span>
                <span>-{{ getCurrencySymbol(invoice.currency) }}{{ invoice.discountAmount.toFixed(2) }}</span>
              </div>
              <div class="total-line" v-if="invoice.taxRate > 0">
                <span>Tax ({{ invoice.taxRate }}%):</span>
                <span>{{ getCurrencySymbol(invoice.currency) }}{{ invoice.taxAmount.toFixed(2) }}</span>
              </div>
              <div class="total-line grand-total">
                <span>Total:</span>
                <span>{{ getCurrencySymbol(invoice.currency) }}{{ invoice.total.toFixed(2) }}</span>
              </div>
            </div>
            
            <div class="preview-notes" v-if="invoice.notes">
              <h4>Notes:</h4>
              <p>{{ invoice.notes }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Email Modal -->
    <div class="modal" v-if="showEmailModal">
      <div class="modal-content">
        <span class="close" @click="showEmailModal = false">&times;</span>
        <h2>Send Invoice via Email</h2>
        <div class="form-group">
          <label for="emailTo">To:</label>
          <input type="email" id="emailTo" v-model="emailForm.to" :placeholder="invoice.clientEmail || 'client@example.com'" />
        </div>
        <div class="form-group">
          <label for="emailSubject">Subject:</label>
          <input type="text" id="emailSubject" v-model="emailForm.subject" placeholder="Invoice from Your Company" />
        </div>
        <div class="form-group">
          <label for="emailMessage">Message:</label>
          <textarea id="emailMessage" v-model="emailForm.message" placeholder="Please find attached the invoice for your recent purchase."></textarea>
        </div>
        <div class="modal-actions">
          <button @click="sendEmail" class="btn primary">Send</button>
          <button @click="showEmailModal = false" class="btn secondary">Cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import html2pdf from 'html2pdf.js';

export default {
  name: 'InvoiceGenerator',
  setup() {
    const previewContainer = ref(null);
    const showEmailModal = ref(false);
    
    const invoice = reactive({
      companyName: '',
      companyAddress: '',
      companyEmail: '',
      companyPhone: '',
      companyLogo: null,
      
      clientName: '',
      clientAddress: '',
      clientEmail: '',
      clientPhone: '',
      
      invoiceNumber: 'INV-' + new Date().getFullYear() + '-' + String(Math.floor(Math.random() * 1000)).padStart(3, '0'),
      invoiceDate: new Date().toISOString().split('T')[0],
      dueDate: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
      
      items: [{ description: '', quantity: 1, price: 0 }],
      
      taxRate: 0,
      discount: 0,
      
      subtotal: 0,
      discountAmount: 0,
      taxAmount: 0,
      total: 0,
      
      currency: 'USD',
      notes: ''
    });
    
    const emailForm = reactive({
      to: '',
      subject: 'Invoice from ' + (invoice.companyName || 'Your Company'),
      message: 'Please find attached the invoice for your recent purchase.'
    });
    
    const calculateItemTotal = (item) => {
      const quantity = item.quantity || 0;
      const price = item.price || 0;
      return quantity * price;
    };

    const calculateTotals = () => {
      invoice.subtotal = invoice.items.reduce((sum, item) => {
        const quantity = item.quantity || 0;
        const price = item.price || 0;
        return sum + (quantity * price);
      }, 0);
      invoice.discountAmount = invoice.subtotal * ((invoice.discount || 0) / 100);
      const afterDiscount = invoice.subtotal - invoice.discountAmount;
      invoice.taxAmount = afterDiscount * ((invoice.taxRate || 0) / 100);
      invoice.total = afterDiscount + invoice.taxAmount;
    };
    
    const addItem = () => {
      invoice.items.push({ description: '', quantity: 1, price: 0 });
    };
    
    const removeItem = (index) => {
      if (invoice.items.length > 1) {
        invoice.items.splice(index, 1);
        calculateTotals();
      }
    };
    
    const onSelectLogo = (event) => {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          invoice.companyLogo = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    };
    
    const onDropLogo = (event) => {
      const file = event.dataTransfer.files[0];
      if (file && file.type.match('image.*')) {
        const reader = new FileReader();
        reader.onload = (e) => {
          invoice.companyLogo = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    };
    
    const removeLogo = () => {
      invoice.companyLogo = null;
    };
    
    const formatDate = (dateString) => {
      if (!dateString) return '';
      const date = new Date(dateString);
      return date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
    };
    
    const getCurrencySymbol = (currency) => {
      const symbols = {
        USD: '$',
        EUR: '€',
        GBP: '£',
        JPY: '¥',
        CAD: '$',
        AUD: '$'
      };
      return symbols[currency] || '$';
    };
    
    const generatePDF = () => {
      const element = previewContainer.value;
      const opt = {
        margin: [10, 10, 10, 10],
        filename: `Invoice-${invoice.invoiceNumber}.pdf`,
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
      };
      
      html2pdf().set(opt).from(element).save();
    };
    
    const sendEmail = () => {
      // In a real application, you would integrate with an email service API here
      // For this example, we'll just show an alert
      alert(`Email would be sent to: ${emailForm.to || invoice.clientEmail}\nSubject: ${emailForm.subject}\nMessage: ${emailForm.message}`);
      showEmailModal.value = false;
    };
    
    onMounted(() => {
      calculateTotals();
    });
    
    return {
      invoice,
      previewContainer,
      showEmailModal,
      emailForm,
      calculateTotals,
      calculateItemTotal,
      addItem,
      removeItem,
      onSelectLogo,
      onDropLogo,
      removeLogo,
      formatDate,
      getCurrencySymbol,
      generatePDF,
      sendEmail
    };
  }
}
</script>

<style>
:root {
  --primary-color: #5b21b6;
  --primary-hover: #4c1d95;
  --secondary-color: #2dd4bf;
  --secondary-hover: #14b8a6;
  --light-color: #f8fafc;
  --dark-color: #1e293b;
  --border-color: #e2e8f0;
  --border-radius: 8px;
  --box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  --transition: all 0.3s ease;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Inter', 'Segoe UI', system-ui, -apple-system, sans-serif;
  line-height: 1.6;
  color: #334155;
  background-color: #f1f5f9;
}

.invoice-generator {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
}

header {
  text-align: center;
  margin-bottom: 30px;
}

header h1 {
  color: var(--primary-color);
  font-size: 2.5rem;
  font-weight: 800;
  position: relative;
  display: inline-block;
  margin-bottom: 0.5rem;
}

header h1::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 4px;
  background-color: var(--secondary-color);
  border-radius: 2px;
}

.container {
  background-color: white;
  border-radius: 12px;
  box-shadow: var(--box-shadow);
  overflow: hidden;
  margin-bottom: 40px;
}

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 24px;
  background-color: #f8fafc;
  border-bottom: 1px solid var(--border-color);
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: var(--border-radius);
  cursor: pointer;
  font-weight: 600;
  transition: var(--transition);
  font-size: 0.95rem;
}

.btn.primary {
  background-color: var(--primary-color);
  color: white;
}

.btn.primary:hover {
  background-color: var(--primary-hover);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.btn.secondary {
  background-color: var(--secondary-color);
  color: white;
}

.btn.secondary:hover {
  background-color: var(--secondary-hover);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.currency-selector {
  display: flex;
  align-items: center;
  gap: 10px;
}

.currency-selector label {
  font-weight: 600;
  color: #64748b;
}

.currency-selector select {
  padding: 10px;
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  background-color: white;
  font-weight: 500;
  color: #334155;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  transition: var(--transition);
}

.currency-selector select:focus {
  outline: none;
  border-color: var(--secondary-color);
  box-shadow: 0 0 0 3px rgba(45, 212, 191, 0.2);
}

.invoice-form-container {
  display: flex;
  flex-wrap: wrap;
}

.invoice-form {
  flex: 1;
  min-width: 300px;
  padding: 24px;
  border-right: 1px solid var(--border-color);
}

.invoice-preview {
  flex: 1;
  min-width: 300px;
  padding: 24px;
  background-color: #f8fafc;
}

.form-section {
  margin-bottom: 32px;
  background-color: #fff;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

.form-section h2 {
  margin-bottom: 18px;
  color: var(--primary-color);
  font-size: 1.25rem;
  font-weight: 700;
  border-bottom: 2px solid var(--border-color);
  padding-bottom: 10px;
}

.form-row {
  display: flex;
  flex-wrap: wrap;
  gap: 18px;
  margin-bottom: 18px;
}

.form-group {
  flex: 1;
  min-width: 200px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: 600;
  color: #475569;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 12px;
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  font-family: inherit;
  transition: var(--transition);
  font-size: 0.95rem;
  color: #334155;
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
  outline: none;
  border-color: var(--secondary-color);
  box-shadow: 0 0 0 3px rgba(45, 212, 191, 0.2);
}

.form-group textarea {
  min-height: 100px;
  resize: vertical;
}

.logo-upload {
  border: 2px dashed #cbd5e1;
  border-radius: var(--border-radius);
  padding: 24px;
  text-align: center;
  margin-bottom: 20px;
  min-height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
  background-color: #f8fafc;
}

.logo-upload:hover {
  border-color: var(--secondary-color);
  background-color: #f0fdfa;
}

.logo-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  color: #64748b;
}

.upload-icon {
  font-size: 2.5rem;
  color: #94a3b8;
}

.upload-btn {
  background-color: #e0f2fe;
  padding: 8px 16px;
  border-radius: var(--border-radius);
  cursor: pointer;
  color: var(--primary-color);
  font-weight: 600;
  transition: var(--transition);
}

.upload-btn:hover {
  background-color: #bae6fd;
}

.hidden-input {
  display: none;
}

.logo-preview {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
}

.logo-preview img {
  max-width: 100%;
  max-height: 120px;
  object-fit: contain;
  border-radius: 4px;
}

.remove-logo {
  position: absolute;
  top: -10px;
  right: -10px;
  background-color: #ef4444;
  color: white;
  border: none;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 16px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: var(--transition);
}

.remove-logo:hover {
  background-color: #dc2626;
  transform: scale(1.1);
}

.items-table {
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  overflow: hidden;
  margin-bottom: 18px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.items-header {
  display: grid;
  grid-template-columns: 3fr 1fr 1fr 1fr 40px;
  background-color: #f1f5f9;
  padding: 12px;
  font-weight: 600;
  color: #475569;
}

.item-row {
  display: grid;
  grid-template-columns: 3fr 1fr 1fr 1fr 40px;
  padding: 12px;
  border-top: 1px solid var(--border-color);
  background-color: white;
  transition: var(--transition);
}

.item-row:hover {
  background-color: #f8fafc;
}

.item-row input {
  width: 100%;
  padding: 8px 10px;
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  transition: var(--transition);
}

.item-row input:focus {
  outline: none;
  border-color: var(--secondary-color);
  box-shadow: 0 0 0 3px rgba(45, 212, 191, 0.2);
}

.btn-remove {
  background-color: #ef4444;
  color: white;
  border: none;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: var(--transition);
  font-size: 16px;
}

.btn-remove:hover {
  background-color: #dc2626;
  transform: scale(1.1);
}

.add-item {
  background-color: #f0fdfa;
  color: var(--primary-color);
  border: 1px dashed var(--secondary-color);
  display: block;
  width: 100%;
  font-weight: 600;
  margin-top: 10px;
  padding: 10px;
}

.add-item:hover {
  background-color: #ccfbf1;
  border-color: var(--primary-color);
}

.totals-container {
  background-color: #f8fafc;
  padding: 20px;
  border-radius: var(--border-radius);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.totals-summary {
  margin-top: 18px;
}

.total-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
}

.grand-total {
  font-weight: bold;
  font-size: 1.2rem;
  border-top: 2px solid var(--border-color);
  padding-top: 12px;
  margin-top: 8px;
  color: var(--primary-color);
}

/* Preview Styles */
.preview-container {
  background-color: white;
  padding: 36px;
  border-radius: var(--border-radius);
  box-shadow: var(--box-shadow);
}

.invoice-preview h2 {
  color: var(--primary-color);
  margin-bottom: 20px;
  font-weight: 700;
  position: relative;
  display: inline-block;
}

.invoice-preview h2::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 0;
  width: 40px;
  height: 3px;
  background-color: var(--secondary-color);
  border-radius: 2px;
}

.preview-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 36px;
}

.preview-logo img {
  max-width: 150px;
  max-height: 80px;
}

.preview-company h3 {
  color: var(--primary-color);
  margin-bottom: 5px;
  font-weight: 700;
}

.preview-invoice-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 36px;
}

.preview-invoice-details h2 {
  color: var(--primary-color);
  font-size: 2.5rem;
  font-weight: 800;
  letter-spacing: 1px;
}

.preview-client {
  margin-bottom: 36px;
  padding: 20px;
  background-color: #f8fafc;
  border-radius: var(--border-radius);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.preview-client h4 {
  color: var(--primary-color);
  margin-bottom: 8px;
  font-weight: 600;
}

.preview-items table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 36px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.preview-items th {
  background-color: #f1f5f9;
  padding: 12px;
  text-align: left;
  border-bottom: 2px solid var(--border-color);
  color: #475569;
  font-weight: 600;
}

.preview-items td {
  padding: 12px;
  border-bottom: 1px solid var(--border-color);
}

.preview-items tr:nth-child(even) {
  background-color: #f8fafc;
}

.preview-totals {
  margin-left: auto;
  width: 300px;
  padding: 20px;
  background-color: #f8fafc;
  border-radius: var(--border-radius);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.total-line {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
}

.preview-notes {
  margin-top: 36px;
  padding: 20px;
  background-color: #f8fafc;
  border-radius: var(--border-radius);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.preview-notes h4 {
  color: var(--primary-color);
  margin-bottom: 8px;
  font-weight: 600;
}

/* Modal Styles */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
}

.modal-content {
  background-color: white;
  padding: 36px;
  border-radius: 12px;
  width: 500px;
  max-width: 90%;
  position: relative;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
}

.close {
  position: absolute;
  top: 15px;
  right: 20px;
  font-size: 1.8rem;
  cursor: pointer;
  color: #64748b;
  transition: var(--transition);
}

.close:hover {
  color: #ef4444;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

/* Improved Responsive Styles */
@media (max-width: 1200px) {
  .invoice-form-container {
    flex-direction: column;
  }
  
  .invoice-form {
    border-right: none;
    border-bottom: 1px solid var(--border-color);
  }
  
  .preview-header {
    flex-direction: column;
    gap: 20px;
  }
  
  .preview-company {
    text-align: left;
  }
}

@media (max-width: 768px) {
  .invoice-generator {
    padding: 15px;
  }
  
  header h1 {
    font-size: 2rem;
  }
  
  .controls {
    flex-wrap: wrap;
    gap: 10px;
  }
  
  .controls .btn {
    flex: 1;
    min-width: 120px;
    text-align: center;
  }
  
  .currency-selector {
    width: 100%;
    justify-content: flex-start;
    margin-top: 10px;
  }
  
  .form-section {
    padding: 15px;
    max-width: 400px;
  }

  
  .items-header, .item-row {
    grid-template-columns: 3fr 1fr 1fr 40px;
  }
  
  .item-price {
    display: none;
  }
  
  .preview-container {
    padding: 20px;
  }
  
  .preview-invoice-details {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }
  
  .preview-totals {
    width: 100%;
  }
}

@media (max-width: 480px) {
  .controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .form-row {
    flex-direction: column;
    gap: 12px;
  }

  .form-section {
    padding: 15px;
    max-width: 300px;
  }

  .invoice-preview {
    max-width: 322px;
  }
  
  .items-header, .item-row {
    grid-template-columns: 3fr 1fr 40px;
    font-size: 14px;
  }
  
  .item-total {
    display: none;
  }

  .preview-items th, .preview-items td {
    padding: 4px;
    font-size: 12px;
  }
  
  .modal-content {
    padding: 20px;
  }
}

/* Fix for even smaller screens */
@media (max-width: 360px) {
  .items-header, .item-row {
    grid-template-columns: 2fr 1fr 40px;
    font-size: 12px;
  }
  
  .btn {
    padding: 8px 16px;
    font-size: 14px;
  }
  .form-section {
    padding: 15px;
    max-width: 270px;
  }
}
</style>