# MVP1 Payment Processing (Test Mode Only)

## MVP1 Scope: Test Mode Implementation
**CRITICAL**: This is TEST MODE ONLY. Production payment processing comes in MVP2.

## MVP1 Payment Flow
```
STUDENT FLOW (MVP1 - Test Mode):
1. Student sees fee structure with GST breakdown
2. Student clicks "Pay Now" → Redirected to TEST Razorpay
3. Student enters TEST card details (4111 1111 1111 1111)
4. Payment succeeds in TEST mode
5. System creates TEST transaction record
6. Student receives TEST receipt (not real money)
```

## MVP1 Database Schema
```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  studentId: ObjectId (ref: students),
  admissionId: ObjectId (ref: admissions),
  amount: Number, // Test amount only
  currency: String (default: "INR"),
  razorpayOrderId: String, // Test order ID
  razorpayPaymentId: String, // Test payment ID
  status: String (enum: ["PENDING", "SUCCESS", "FAILED"]), // Test status
  idempotencyKey: String (unique per transaction), // MVP1: Critical for test
  receiptUrl: String, // Test receipt URL
  createdAt: Date,
  updatedAt: Date
}
```

## MVP1 Implementation Code
```javascript
// MVP1: Test Mode Payment Creation
const createTestPaymentSession = async (req, res) => {
  try {
    const { amount, currency, studentId, admissionId } = req.body;
    const collegeId = req.college._id;

    // MVP1: Generate idempotency key to prevent test duplicates
    const idempotencyKey = `test-${collegeId}-${studentId}-${Date.now()}`;
    
    // MVP1: Check if test transaction already exists
    const existing = await Transaction.findOne({ idempotencyKey });
    if (existing) {
      return res.json({
        sessionId: existing.razorpayOrderId,
        transactionId: existing._id,
        testMode: true
      });
    }

    // MVP1: Create TEST Razorpay order (using test keys)
    const testOrder = {
      id: `order_test_${Date.now()}`,
      amount: amount * 100, // in paise
      currency: currency || 'INR',
      status: 'created'
    };

    // MVP1: Save test transaction
    const transaction = await Transaction.create({
      collegeId,
      studentId,
      admissionId,
      amount,
      razorpayOrderId: testOrder.id,
      status: 'PENDING',
      idempotencyKey,
      createdAt: new Date()
    });

    res.json({
      orderId: testOrder.id,
      transactionId: transaction._id,
      testMode: true,
      testAmount: testOrder.amount
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

## MVP1 Limitations & Future (MVP2)
- **MVP1**: Test mode only (no real money)
- **MVP2**: Production payment processing with webhooks
- **MVP1**: No automatic receipt generation (manual in test)
- **MVP2**: Automated receipt with proper GST compliance
- **MVP1**: No failed payment recovery (manual in test)
- **MVP2**: Automated retry and recovery system