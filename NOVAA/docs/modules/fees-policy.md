# Fees & Policy Module Implementation

## MVP1 Fee Management (Test Mode)

### Core Features
- Fee structure configuration per course
- GST calculation for different fee types
- Test-mode payment processing
- Fee receipt generation (manual in MVP1)

### Fee Structure Schema
```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  courseId: String,
  tuitionFee: Number,     // GST exempt (educational)
  labFee: Number,         // 18% GST applicable
  sportsFee: Number,      // 18% GST applicable
  libraryFee: Number,     // 18% GST applicable
  transportFee: Number,   // 5% GST applicable
  admissionFee: Number,   // May vary by category
  effectiveDate: Date,
  isActive: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

### GST Calculation Logic
```javascript
const GST_RATES = {
  TUITION: 0.00,        // Educational - no tax
  LAB: 0.18,            // 18% GST
  SPORTS: 0.18,         // 18% GST
  LIBRARY: 0.18,        // 18% GST
  TRANSPORT: 0.05,      // 5% GST
  ADMISSION: 0.00       // Service - may vary
};

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

### Fee Structure Configuration
```javascript
const configureFeeStructure = async (req, res) => {
  try {
    const {
      courseId,
      tuitionFee,
      labFee,
      sportsFee,
      libraryFee,
      transportFee,
      admissionFee
    } = req.body;
    
    const collegeId = req.college._id;

    // Validate fee amounts
    if (tuitionFee < 0 || labFee < 0 || sportsFee < 0) {
      return res.status(400).json({ error: 'Fees cannot be negative' });
    }

    // Calculate GST breakdown
    const gstBreakdown = GSTCalculator.calculateTotal({
      tuition: tuitionFee,
      lab: labFee,
      sports: sportsFee,
      library: libraryFee,
      transport: transportFee,
      admission: admissionFee
    });

    // Create fee structure
    const feeStructure = await FeeStructure.create({
      collegeId,
      courseId,
      tuitionFee,
      labFee,
      sportsFee,
      libraryFee,
      transportFee,
      admissionFee,
      gstBreakdown,
      effectiveDate: new Date(),
      isActive: true
    });

    res.status(201).json({
      message: 'Fee structure created successfully',
      feeStructure
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Fee Display for Students
```javascript
const getStudentFeeDetails = async (req, res) => {
  try {
    const { courseId } = req.params;
    const collegeId = req.college._id;

    // Get active fee structure
    const feeStructure = await FeeStructure.findOne({
      collegeId,
      courseId,
      isActive: true
    }).sort({ effectiveDate: -1 });

    if (!feeStructure) {
      return res.status(404).json({ error: 'Fee structure not found' });
    }

    // Calculate GST breakdown for display
    const gstBreakdown = GSTCalculator.calculateTotal({
      tuition: feeStructure.tuitionFee,
      lab: feeStructure.labFee,
      sports: feeStructure.sportsFee,
      library: feeStructure.libraryFee,
      transport: feeStructure.transportFee,
      admission: feeStructure.admissionFee
    });

    res.json({
      courseId,
      fees: {
        tuition: feeStructure.tuitionFee,
        lab: feeStructure.labFee,
        sports: feeStructure.sportsFee,
        library: feeStructure.libraryFee,
        transport: feeStructure.transportFee,
        admission: feeStructure.admissionFee
      },
      gstBreakdown,
      totalAmount: gstBreakdown.total
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### MVP1 Limitations & Future (MVP2)
- **MVP1**: Test-mode fee calculations only
- **MVP2**: Production fee processing with real payments
- **MVP1**: Manual receipt generation
- **MVP2**: Automated receipt with proper GST compliance
- **MVP1**: Basic fee structure only
- **MVP2**: Advanced fee policies and scholarships