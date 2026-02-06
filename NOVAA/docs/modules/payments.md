# Payments Module Implementation

## MVP1 Payment Processing (Test Mode)

### Fee Structure & GST Calculation

#### Fee Types
```javascript
// Fee Categories with GST Rates
const GST_RATES = {
  TUITION: 0.00,        // Educational - no tax
  LAB: 0.18,            // 18% GST
  SPORTS: 0.18,         // 18% GST
  LIBRARY: 0.18,        // 18% GST
  TRANSPORT: 0.05,      // 5% GST
  ADMISSION: 0.00       // Service - may vary
};
```

#### GST Calculator
```javascript
class GSTCalculator {
  static calculateGST(feeType, amount) {
    const rate = GST_RATES[feeType.toUpperCase()];
    if (rate === undefined) {
      throw new Error(`Unknown fee type: ${feeType}`);
    }
    return Math.round(amount * rate);
  }

  static calculateTotal(fees) {
    let subtotal = 0;
    let totalGST = 0;

    for (const [feeType, amount] of Object.entries(fees)) {
      const gst = this.calculateGST(feeType, amount);
      subtotal += amount;
      totalGST += gst;
    }

    return {
      subtotal,
      totalGST,
      total: subtotal + totalGST
    };
  }
}
```

### Razorpay Integration (Test Mode)

#### Setup
```javascript
const Razorpay = require('razorpay');

const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,      // MVP1: Test key
  key_secret: process.env.RAZORPAY_KEY_SECRET  // MVP1: Test secret
});
```

#### Order Creation
```javascript
const createTestOrder = async (req, res) => {
  try {
    const { studentId, admissionId, amount } = req.body;
    const collegeId = req.user.collegeId;

    // MVP1: Generate idempotency key for test mode
    const idempotencyKey = `test-${collegeId}-${studentId}-${admissionId}-${Date.now()}`;

    // MVP1: Check if order already exists (test mode duplicate prevention)
    const existing = await Transaction.findOne({ idempotencyKey });
    if (existing) {
      return res.status(200).json({
        message: 'Using existing test order',
        orderId: existing.razorpayOrderId,
        transactionId: existing._id
      });
    }

    // MVP1: Create test Razorpay order
    const testOrder = {
      id: `order_test_${Date.now()}`,
      amount: amount * 100,        // Razorpay expects paise
      currency: 'INR',
      receipt: `test-admission-${admissionId}`,
      notes: {
        studentId,
        admissionId,
        collegeId: collegeId.toString(),
        testMode: true
      }
    };

    // MVP1: Save test transaction
    const transaction = await Transaction.create({
      studentId,
      admissionId,
      collegeId,
      idempotencyKey,
      razorpayOrderId: testOrder.id,
      amount,
      status: 'ORDER_CREATED',
      testMode: true
    });

    res.status(201).json({
      orderId: testOrder.id,
      amount: testOrder.amount,
      currency: testOrder.currency,
      transactionId: transaction._id,
      testMode: true
    });

  } catch (error) {
    console.error('Test order creation error:', error);
    res.status(500).json({ error: 'Failed to create test order' });
  }
};
```

### MVP1 Limitations
- **Test Mode Only**: No real money transactions
- **No Webhooks**: Manual payment status checking
- **No Receipt Generation**: Will be implemented in MVP2
- **No Failed Payment Recovery**: Manual handling in MVP1