# NOVAA MVP1 - Current Implementation Status & Payment Integration Guide

## Overview
This document provides a comprehensive overview of the current implementation status of NOVAA MVP1, identifies gaps between planned and actual implementation, and provides detailed guidance on integrating both Stripe and Razorpay payment gateways.

## Current Implementation Status

### ✅ Completed Features
1. **Authentication System** - JWT-based authentication with role-based access control
2. **College Management** - Multi-tenancy with data isolation
3. **Admissions Module** - Student application and verification workflow
4. **Manual Attendance** - Basic attendance tracking system
5. **Basic Reports** - Read-only reporting APIs
6. **Internal Notifications** - Communication system
7. **Payment Processing** - Stripe integration (test mode only)

### ⚠️ Current Limitations & Gaps

#### 1. Payment System Gap
- **Planned**: Both Stripe and Razorpay integration
- **Current**: Stripe integration only (test mode)
- **Issue**: Razorpay integration not implemented
- **Impact**: Limited payment options for educational organizations

#### 2. Attendance System Gap
- **Planned**: QR-based attendance system
- **Current**: Manual attendance only
- **Issue**: No automated attendance tracking
- **Impact**: Higher administrative overhead

#### 3. Payment Processing Limitations
- **Planned**: Production payment processing with webhooks
- **Current**: Test mode only, no webhook integration
- **Issue**: Payment confirmations rely on frontend redirects
- **Impact**: Potential for failed payment confirmations

#### 4. Reporting System Limitations
- **Planned**: Export functionality (PDF/Excel)
- **Current**: Backend APIs only
- **Issue**: No export capabilities
- **Impact**: Limited reporting flexibility

## Payment Gateway Integration Guide

### Dual Payment Gateway Strategy
NOVAA will offer both Stripe and Razorpay to accommodate different educational organizations' preferences and regional requirements.

### Stripe Integration

#### Installation & Setup
```bash
# Install Stripe SDK
npm install stripe

# Environment variables
STRIPE_PUBLISHABLE_KEY=pk_test_your_publishable_key
STRIPE_SECRET_KEY=sk_test_your_secret_key
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret
STRIPE_SUCCESS_URL=https://yourdomain.com/payment-success
STRIPE_CANCEL_URL=https://yourdomain.com/payment-cancel
```

#### Backend Implementation
```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// Create payment session
const createPaymentSession = async (req, res) => {
  try {
    const { amount, currency, studentId, feeStructureId } = req.body;
    
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card', 'upi'],
      line_items: [{
        price_data: {
          currency: currency || 'inr',
          product_data: {
            name: 'College Fee Payment',
            description: 'Fee payment for educational services'
          },
          unit_amount: amount * 100, // Convert to cents
        },
        quantity: 1,
      }],
      mode: 'payment',
      success_url: process.env.STRIPE_SUCCESS_URL + '?session_id={CHECKOUT_SESSION_ID}',
      cancel_url: process.env.STRIPE_CANCEL_URL,
      client_reference_id: studentId,
      metadata: {
        studentId,
        feeStructureId,
        collegeId: req.college._id
      }
    });

    res.json({ sessionId: session.id });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Webhook handler
app.post('/webhooks/stripe', express.raw({type: 'application/json'}), (req, res) => {
  const sig = req.headers['stripe-signature'];
  const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET;

  let event;

  try {
    event = stripe.webhooks.constructEvent(req.body, sig, endpointSecret);
  } catch (err) {
    console.log(`Webhook signature verification failed.`, err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // Handle the event
  switch (event.type) {
    case 'payment_intent.succeeded':
      const paymentIntent = event.data.object;
      // Update transaction status to SUCCESS
      // Generate receipt
      // Send notifications
      break;
    case 'payment_intent.payment_failed':
      const paymentFailure = event.data.object;
      // Update transaction status to FAILED
      // Add to failed payments queue
      break;
    case 'checkout.session.completed':
      const session = event.data.object;
      // Finalize enrollment after payment
      break;
    default:
      console.log(`Unhandled event type ${event.type}`);
  }

  res.json({received: true});
});
```

### Razorpay Integration

#### Installation & Setup
```bash
# Install Razorpay SDK
npm install razorpay

# Environment variables
RAZORPAY_KEY_ID=rzp_test_your_key_id
RAZORPAY_KEY_SECRET=your_key_secret
RAZORPAY_WEBHOOK_SECRET=your_webhook_secret
```

#### Backend Implementation
```javascript
const Razorpay = require('razorpay');
const crypto = require('crypto');

// Initialize Razorpay instance
const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET
});

// Create order
const createRazorpayOrder = async (req, res) => {
  try {
    const { amount, currency, studentId, feeStructureId } = req.body;
    
    const options = {
      amount: amount * 100, // Amount in paise
      currency: currency || 'INR',
      receipt: `order_${Date.now()}_${studentId}`,
      payment_capture: 1 // Auto-capture payment
    };

    const order = await razorpay.orders.create(options);
    
    res.json({
      orderId: order.id,
      amount: order.amount,
      currency: order.currency,
      key: process.env.RAZORPAY_KEY_ID
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Verify payment
const verifyRazorpayPayment = async (req, res) => {
  try {
    const { razorpay_order_id, razorpay_payment_id, razorpay_signature, studentId } = req.body;

    // Create signature verification string
    const body = razorpay_order_id + "|" + razorpay_payment_id;
    const expectedSignature = crypto
      .createHmac('sha256', process.env.RAZORPAY_KEY_SECRET)
      .update(body.toString())
      .digest('hex');

    // Verify signature
    if (expectedSignature === razorpay_signature) {
      // Payment verified successfully
      // Update transaction status to SUCCESS
      // Generate receipt
      // Send notifications
      
      res.json({ success: true, paymentId: razorpay_payment_id });
    } else {
      res.status(400).json({ success: false, error: 'Payment verification failed' });
    }
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Webhook handler
app.post('/webhooks/razorpay', express.raw({type: 'application/json'}), (req, res) => {
  try {
    const secret = process.env.RAZORPAY_WEBHOOK_SECRET;
    
    // Verify webhook signature
    const signature = req.headers['x-razorpay-signature'];
    const shasum = crypto.createHmac('sha256', secret);
    shasum.update(req.body.toString());
    const digest = shasum.digest('hex');

    if (digest !== signature) {
      return res.status(400).json({ error: 'Invalid signature' });
    }

    const event = JSON.parse(req.body);

    // Handle different event types
    switch (event.event) {
      case 'payment.captured':
        const payment = event.payload.payment.entity;
        // Update transaction status to SUCCESS
        // Generate receipt
        // Send notifications
        break;
      case 'payment.failed':
        const failure = event.payload.payment.entity;
        // Update transaction status to FAILED
        // Add to failed payments queue
        break;
      case 'order.paid':
        const order = event.payload.order.entity;
        // Finalize enrollment after payment
        break;
      default:
        console.log(`Unhandled event type ${event.event}`);
    }

    res.json({ received: true });
  } catch (error) {
    console.error('Webhook verification failed:', error);
    res.status(500).json({ error: error.message });
  }
});
```

### Unified Payment Controller

```javascript
class PaymentController {
  static async createPaymentSession(req, res) {
    const { amount, currency, paymentGateway, studentId, feeStructureId } = req.body;
    
    try {
      if (paymentGateway === 'stripe') {
        // Use Stripe implementation
        const session = await stripe.checkout.sessions.create({
          payment_method_types: ['card', 'upi'],
          line_items: [{
            price_data: {
              currency: currency || 'inr',
              product_data: {
                name: 'College Fee Payment',
                description: 'Fee payment for educational services'
              },
              unit_amount: amount * 100,
            },
            quantity: 1,
          }],
          mode: 'payment',
          success_url: process.env.STRIPE_SUCCESS_URL + '?session_id={CHECKOUT_SESSION_ID}',
          cancel_url: process.env.STRIPE_CANCEL_URL,
          client_reference_id: studentId,
          metadata: {
            studentId,
            feeStructureId,
            collegeId: req.college._id,
            paymentGateway: 'stripe'
          }
        });
        
        res.json({ 
          sessionId: session.id, 
          paymentGateway: 'stripe',
          paymentUrl: session.url 
        });
      } else if (paymentGateway === 'razorpay') {
        // Use Razorpay implementation
        const options = {
          amount: amount * 100,
          currency: currency || 'INR',
          receipt: `order_${Date.now()}_${studentId}`,
          payment_capture: 1
        };

        const order = await razorpay.orders.create(options);
        
        res.json({ 
          orderId: order.id, 
          paymentGateway: 'razorpay',
          amount: order.amount,
          currency: order.currency,
          key: process.env.RAZORPAY_KEY_ID
        });
      } else {
        res.status(400).json({ error: 'Invalid payment gateway' });
      }
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  static async handlePaymentSuccess(req, res) {
    const { paymentGateway } = req.body;
    
    if (paymentGateway === 'stripe') {
      // Handle Stripe success
      await this.handleStripeSuccess(req, res);
    } else if (paymentGateway === 'razorpay') {
      // Handle Razorpay success
      await this.handleRazorpaySuccess(req, res);
    }
  }
}
```

### Frontend Integration

```javascript
// Payment selection component
const PaymentSelection = ({ amount, feeStructureId }) => {
  const [selectedGateway, setSelectedGateway] = useState('stripe');
  const [loading, setLoading] = useState(false);

  const initiatePayment = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/payments/create-session', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          amount,
          paymentGateway: selectedGateway,
          feeStructureId
        })
      });

      const data = await response.json();

      if (selectedGateway === 'stripe') {
        // Redirect to Stripe checkout
        window.location.href = data.paymentUrl;
      } else if (selectedGateway === 'razorpay') {
        // Open Razorpay checkout
        const options = {
          key: data.key,
          amount: data.amount,
          currency: data.currency,
          name: 'NOVAA College Payment',
          description: 'Fee payment for educational services',
          order_id: data.orderId,
          handler: async function (response) {
            // Verify payment
            const verifyResponse = await fetch('/api/payments/verify', {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
              },
              body: JSON.stringify(response)
            });
            
            if (verifyResponse.ok) {
              alert('Payment successful!');
              window.location.href = '/payment-success';
            }
          },
          prefill: {
            name: 'Customer Name',
            email: 'customer@example.com',
            contact: '9999999999'
          },
          theme: {
            color: '#3399cc'
          }
        };
        
        const rzp = new window.Razorpay(options);
        rzp.open();
      }
    } catch (error) {
      console.error('Payment initiation failed:', error);
      alert('Payment initiation failed. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="payment-selection">
      <h3>Select Payment Method</h3>
      <div className="gateway-options">
        <label>
          <input
            type="radio"
            value="stripe"
            checked={selectedGateway === 'stripe'}
            onChange={() => setSelectedGateway('stripe')}
          />
          Stripe (Credit/Debit Cards, UPI)
        </label>
        <label>
          <input
            type="radio"
            value="razorpay"
            checked={selectedGateway === 'razorpay'}
            onChange={() => setSelectedGateway('razorpay')}
          />
          Razorpay (Cards, Wallets, UPI)
        </label>
      </div>
      <button onClick={initiatePayment} disabled={loading}>
        {loading ? 'Processing...' : `Pay ₹${amount}`}
      </button>
    </div>
  );
};
```

### Webhook Configuration

#### Stripe Webhook Setup
1. Go to Stripe Dashboard → Developers → Webhooks
2. Click "Add endpoint"
3. Enter your webhook URL: `https://yourdomain.com/webhooks/stripe`
4. Select events to listen for:
   - `payment_intent.succeeded`
   - `payment_intent.payment_failed`
   - `checkout.session.completed`
5. Copy the signing secret and add to environment variables

#### Razorpay Webhook Setup
1. Go to Razorpay Dashboard → Settings → Webhooks
2. Add your webhook URL: `https://yourdomain.com/webhooks/razorpay`
3. Select events to listen for:
   - `payment.captured`
   - `payment.failed`
   - `order.paid`
4. Set the secret and add to environment variables

## Implementation Plan for Closing Gaps

### Phase 1: Payment Gateway Enhancement (Week 1)
1. Implement Razorpay integration
2. Set up webhook handlers for both gateways
3. Create unified payment controller
4. Test both payment gateways

### Phase 2: Production Deployment (Week 2)
1. Switch to production credentials
2. Test complete payment flow
3. Implement monitoring for payment confirmations
4. Update documentation

### Phase 3: Advanced Features (Week 3-4)
1. Implement QR-based attendance system
2. Add export functionality to reports
3. Enhance notification system with email/SMS
4. Conduct security audit

## Security Considerations

### Payment Security
- Never store sensitive payment information
- Use HTTPS for all payment-related operations
- Validate all webhook signatures
- Implement rate limiting on payment endpoints
- Use idempotency keys to prevent duplicate charges

### Multi-Tenancy in Payments
- Ensure all payment data is filtered by collegeId
- Validate that students belong to the correct college
- Prevent cross-college payment access
- Implement proper audit logging

## Testing Strategy

### Payment Gateway Testing
- Test both Stripe and Razorpay in test mode
- Verify webhook functionality
- Test payment failure scenarios
- Validate receipt generation
- Test multi-tenancy isolation

### Integration Testing
- End-to-end payment flow testing
- Cross-module integration testing
- Security testing for payment data
- Performance testing for high-volume scenarios

This document provides the roadmap for implementing the missing payment gateway functionality and closing the gaps identified in the current MVP1 implementation.